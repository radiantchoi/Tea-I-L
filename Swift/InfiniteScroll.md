# UIKit `UITableView` 상에서 무한 스크롤 구현할 때 신경써야 할 사항

## 개요: 무한 스크롤이 뭔데?
- 무한히 스크롤(...). 
- 사실 진짜 무한은 아니고, 백엔드에서 불러올 수 있는 자료의 수만큼 보여줄 수 있다.

## 왜 무한 스크롤을 구현하는가?
- 목록 등을 보여줄 때, 서버에 있는 모든 값을 가져와서 앱에 표시하려고 한다면?
- 방대한 데이터를 모두 가져오는 데 시간도 오래 걸리고, 이 데이터를 전부 저장하고 있자니 기기 메모리도 터져나가고, ...
- 그래서, 한 번에 정해진 갯수의 자료만 불러 오고, 사용자가 보여지는 자료의 끝에 다다랐을 때 추가적인 API 콜을 통해 자료를 더 받아 온다.
- 페이지네이션을 사용한다.
  - 클라이언트 사이드에서 API 콜에 포함할 것은 한 번에 불러오는 자료의 갯수, 페이지 번호 등이 있다.
  - 서버 사이드에서는 다음 페이지가 있는지 없는지를 보낼 수 있다.

## `UITableView`의 예시
- 모든 테이블 뷰는 `UIScrollView`를 상속받고, `UITableViewDelegate`를 채택하면 자연히 `UIScrollViewDelegate`도 채택한다.
- `UIScrollViewDelegate`에 있는 `scrollViewDidScroll(_:)` 메서드에서 스크롤을 감지하면 된다.
  - 스크롤을 감지해서, 현재 스크롤 뷰 안에서의 위치와 컨텐츠의 높이를 비교해서, 전자가 더 크다면 컨텐츠가 더 필요하다는 뜻이므로 추가 로드!
- 정확하게 가면, `contentOffset`은 스크롤 뷰의 "Origin"으로부터 스크롤 뷰 안의 컨텐츠 뷰의 "Origin"이 얼마나 떨어져 있는지 나타내는 프로퍼티이다.
  - 그렇기 때문에 컨텐츠 뷰의 오리진이 스크롤 뷰의 높이보다 더 멀리 떨어져 있다면 컨텐츠가 씨가 마른 것.
```Swift
extension CommunityHomeViewController: UITableViewDelegate {
    // ...
    
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let offsetY = postsTableView.contentOffset.y
        let contentHeight = postsTableView.contentSize.height
        let height = postsTableView.frame.size.height
        
        if offsetY > contentHeight - height {
            viewModel.getPosts()
        }
    }
}
```
- 개꿀개이득~ 인 줄 알았지만..

## 무한 스크롤 구현시 주의할 점
- 스크롤을 아래로 땡겼더니 새 자료를 요청하는 API 콜이 와바박 여러 번 들어갔다.
  - 무한 스크롤은 많은 자료를 핸들링하고자 할 때 사용하지만, 역설적으로 자료가 적을 때 이 현상이 일어나기 쉽다.
- 이럴 때를 대비해 로딩 중임을 나타내는 플래그 프로퍼티를 두고, 이것이 가능할 때만 콜하도록 일차적으로 거른다.
- 함수 호출 이벤트 자체를 줄이는 것도 방법이다. 원래는 `delegate` 함수에서 호출하던 것을, 위치를 바꿔 보자.
  - 나는 `Combine` 퍼블리셔의 힘을 빌려 반응형으로 처리해보고자 한다.
```Swift
final class CommunityHomeViewController: BaseViewController {
    // 스크롤 뷰가 스크롤됐을 때 발신하는 Combine 퍼블리셔
    // delegate 함수 업스트림에서 발신된 신호를 받아, 다운스트림으로 발신한다.
    private var scrollPublisher = PassthroughSubject<Void, Never>()
    
    override func observeUIEvents() {
        // ...
        
        // scrollPublisher의 다운스트림
        // throttle을 활용해 일정 주기마다 최초 한 번씩만 이벤트에 반응하여, API 콜을 하도록 동작
        // 물론 이미 구현해 둔 isLoading과 같은 플래그 프로퍼티도 활용
        scrollPublisher
            .throttle(for: 1, scheduler: RunLoop.main, latest: false)
            .sink { [weak self] _ in
                // 뷰 모델에 있는 메서드를 부르는 메서드
                self?.loadMoreData()
            }
            .store(in: &cancellables)
        }
    }
}

extension CommunityHomeViewController: UITableViewDelegate {
    // ...

    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        // 위의 코드와 비교해보면, 스크롤 뷰가 위치가 바뀌었을 때는 그저 신호만 보낸다.
        scrollPublisher.send()
    }
}
```
- 요컨대, 중복 로딩에 신경쓰고, 콜을 한 번에 너무 많이 하지 않도록 신경쓰자.
- 테이블 뷰를 예시로 들었지만, 대개의 UI 프레임워크의 경우 비슷할 것이다.
  - 가령 `SwiftUI`의 경우, `LazyVStack`과 자료 인덱스를 활용해 특정 인덱스의 자료를 그릴 경우 API 콜을 하는 방법을 생각할 수 있겠다.
  - `LazyVStack`은 해당 서브뷰가 표시될 때가 되어서야 화면에 그리기 때문.
  - 아니면 아예 `List`를 쓰던지?! 이거는 좀 더 생각해보자.