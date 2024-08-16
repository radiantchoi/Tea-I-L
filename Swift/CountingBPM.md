# Swift 앱에서 bpm 계산하기

## 발단
- 만들고자 하는 앱에서 일정 주기로 사운드를 재생하는 기능이 필요함
  - 사용자가 특정 bpm을 정하면, 4비트 리듬으로 출력해주는 방식
- `AVFoundation`을 통한 사운드 재생도 복습할 겸..

## Breakdown
- 사용하고자 하는 사운드를 미리 파일로 준비한다.
- bpm == beat per minute.
  - 주어진 숫자를 60으로 나눈다.
  - 60으로 나누었을 때 나오는 숫자가, 1초 동안 재생해야 하는 소리의 수.
  - 예를 들어 120bpm 하에서는 1초에 2번 소리를 재생해야 한다.
  - 그렇다면 소리를 재생하는 코드는 1/2초 주기로 실행되어야 한다.
  - 그렇다면 아예 bpm을 분모로, 60을 분자로 하면 되지 않을까.
- Timer 등의 수단을 활용해, 위에 나온 주기를 토대로 소리를 재생한다.

## 소리 재생을 위한 과정
- 먼저 재생할 소리를 앱 번들에 포함시킨다.
- 다음으로 `AVAudioPlayer`를 세팅한다.
  - 소리를 재생하는 데 중추적인 역할을 한다.
  - `Bundle.main.url`을 통해 앱 번들 내부에 접근해, 미리 저장해 두었던 소리를 가져 온다.
  - 가져온 소리를 바탕을 `AVAudioPlayer` 객체를 만들어 할당하고, `prepareToPlay()` 메서드를 통해 재생에 쓰일 수 있게 한다.
- 그 다음 `Timer`를 세팅한다.
  - 60을 bpm으로 나눈 숫자의 주기로 소리를 재생하게 한다.
  - 온/오프 토글을 통해 타이머를 발동/해제함으로써 소리의 재생을 조절한다.
- 미리 정의한 뷰 및 뷰 모델과 연결해 준다.
  - 뷰 모델에는 재생 중인지 여부와 bpm 값, 빠르기 조절 함수, 재생 토글 함수가 포함된다.
  - 뷰에는 뷰 모델의 메서드와 연결된 버튼을 배치한다.

## 코드

### ContentView
```Swift
import SwiftUI

struct ContentView: View {
    @ObservedObject var viewModel = ContentViewModel()
    
    var body: some View {
        VStack {
            // bpm을 표시하는 UI
            Text("\(viewModel.bpm)")
                .fontWeight(.bold)
                .padding(8)

            // bpm 더하기, 빼기 버튼
            HStack {
                Button {
                    viewModel.decrement()
                } label: {
                    Image(systemName: "minus")
                }
                .padding(8)
                
                Button {
                    viewModel.increment()
                } label: {
                    Image(systemName: "plus")
                }
                .padding(8)
            }
            
            // 재생/일시정지 버튼
            Button {
                viewModel.play()
            } label: {
                Text(viewModel.isPlaying ? "Pause" : "Play")
            }
            .padding(8)
        }
    }
}

#Preview {
    ContentView()
}
```

### ContentViewModel
```Swift
import Foundation

final class ContentViewModel: ObservableObject {
    @Published var bpm: Int = 120
    @Published var isPlaying: Bool = false
    
    // 재생기 객체
    private let beatPlayer = BeatPlayer(soundName: "SmackKick", fileExtension: "wav")
    
    func increment() {
        bpm += 1
    }
    
    func decrement() {
        bpm -= 1
    }
    
    // 세팅되어있는 값에 따라 재생 혹은 일시정지
    func play() {
        if isPlaying {
            isPlaying = false
            beatPlayer.stopPlaying()
        } else {
            isPlaying = true
            beatPlayer.startPlaying(bpm: Double(bpm))
        }
    }
}

```

### BeatPlayer
```Swift
import AVFoundation
import Foundation

final class BeatPlayer {
    private var audioPlayer: AVAudioPlayer?
    private var timer: Timer?
    
    // 초기화시 소리의 이름과 확장자를 입력해, 다양한 소리를 사용할 수 있도록 구현
    init(soundName: String, fileExtension: String) {
        setupAudioPlayer(soundName: soundName, fileExtension: fileExtension)
    }
   
    private func setupAudioPlayer(soundName: String, fileExtension: String) {
        // 먼저 입력받은 정보를 토대로 소리를 찾고
        guard let soundURL = Bundle.main.url(
            forResource: soundName,
            withExtension: fileExtension
        ) else {
            print("음원 파일을 찾을 수 없습니다.")
            return
        }
        
        do {
            // 해당 소리를 사용하는 오디오 플레이어를 세팅한 다음
            audioPlayer = try AVAudioPlayer(contentsOf: soundURL)
            // 재생 준비
            audioPlayer?.prepareToPlay()
        } catch {
            print("오디오 플레이어 설정 중 오류 발생: \(error.localizedDescription)")
        }
    }
    
    func startPlaying(bpm: Double) {
        // 중복 재생을 막기 위한 정지
        stopPlaying()
        
        // 타이머 반복 주기
        let interval = 60.0 / bpm
        
        // 설정된 주기에 따라 소리를 반복하여 재생
        timer = Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { [weak self] _ in
            self?.audioPlayer?.play()
        }
    }
    
    func stopPlaying() {
        // 타이머를 멈추고, 객체를 해제하기
        timer?.invalidate()
        timer = nil
        
        // 동시에 오디오 플레이어도 정지시키고, 타임라인을 0으로 돌리기
        audioPlayer?.stop()
        audioPlayer?.currentTime = 0
    }
    
    deinit {
        // 객체 해제시에도 재생은 멈춰야 한다.
        stopPlaying()
    }
}
```

## 더 알아보면 좋을 키워드
- `AVAudioPlayer.init?`
- `prepareToPlay()`