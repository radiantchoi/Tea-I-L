# Making custom Combine UI event publisher
https://www.avanderlee.com/swift/custom-combine-publisher/
- Swift 반응형 프로그래밍을 수행함에 있어, RxSwift 사용시 UI 이벤트에 대한 구독은 RxCocoa를 통해 수행할 수 있다.
- 하지만 바닐라 Swift 즉, Combine 이용시에는 이런 게 따로 없다.
- 아래의 예제 코드를 보자. 실제로 작동하는 코드이다. (아마도)
```Swift
import Combine
import UIKit

class MyViewController {
    private let viewModel = MyViewModel()
    private var cancellables = Set<AnyCancellable>()
    
    private lazy var stackView: UIStackView = {
        let stackView = UIStackView()
        stackView.translatesAutoresizingMaskIntoConstraints = false
        return stackView
    }()
    
    private lazy var numberLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        return label
    }()
    
    private lazy var incrementButton: UIButton = {
        let button = UIButton(type: .system)
        button.translatesAutoresizingMaskIntoConstraints = false
        button.text = "+1"
        return button
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupView()
        bindViewModel()
    }
    
    private func setupView() {
        view.addSubview(stackView)
        
        stackView.addArrangedSubview(numberLabel)
        stackView.addArrangedSubview(incrementButton)
        
        // 왜 이런 걸 해 주는고 하니..
        incrementButton.addTarget(self, action: #selector(incrementButtonTapped), for: .touchUpInside)
        
        setupConstraints()
    }
    
    private func setupConstraints() {
        stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        stackView.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    }
    
    // 뷰 모델의 published 프로퍼티는 이렇게 컴바인으로 세련되게 받아올 수 있다
    private func bindViewModel() {
        viewModel.number
            .sink { [weak self] n in
                self?.numberLabel.text = String(n)
        }
        .store(in: &cancellables)
    }
    
    // 버튼을 눌렀을 때의 액션은 Obj-C 함수의 힘을 빌려야 한다.
    @objc private func incrementButtonTapped() {
        viewModel.incrementNumber()
    }
} 

final class MyViewModel {
    @Published var number: Int = 0
    
    func incrementNumber() {
        number += 1
    }
}
```
- 위 예제를 보면 버튼에 액션을 더하기 위해 Obj-C API 활용이 있음을 알 수 있다.
- RxCocoa처럼 `incrementButton.rx.tap.bind { }` 이런 식으로 되면 얼마나 좋을까!
- 그렇다면 만들어보자, 커스텀 퍼블리셔!
```Swift
/// UIControl 이벤트를 캡쳐해 구독하는 커스텀 인자.
/// SubscriberType은 Subscriber 즉, 반응형 프로그래밍에서 구독자가 들어가는 파라미터.
/// 구독자는 뭐든지 구독할 수 있기 때문에, Control 인자를 제한함으로써 UIControl만이 발신자가 되게 해 준다.
/// 이는 SubscriberType.Input == UIControl로 됨으로써 이루어진다.
final class UIControlSubscription<SubscriberType: Subscriber, Control: UIControl>: Subscription where SubscriberType.Input == Control {
    private var subscriber: SubscriberType?
    private let control: Control

    init(subscriber: SubscriberType, control: Control, event: UIControl.Event) {
        self.subscriber = subscriber
        self.control = control
        
        // 결국 안에서는 addTarget을 해 줄 수밖에 없지만..
        control.addTarget(self, action: #selector(eventHandler), for: event)
    }

    // Subscribers.Demand 타입은 수신자가 요청하는 이벤트의 갯수를 의미한다.
    // unlimited, none, max(Int) 세 가지 케이스가 있다.
    // 각각의 케이스에 대해 핸들링해야 하지만, 구독을 끊기 전까진 UI 이벤트를 계속 받고 싶으므로 일단 아무것도 하지 않는다.
    func request(_ demand: Subscribers.Demand) {
        // We do nothing here as we only want to send events when they occur.
        // https://developer.apple.com/documentation/combine/subscribers/demand
    }
    
    // 이건 구독 취소고
    func cancel() {
        subscriber = nil
    }

    // 신호를 받았을 떄의 동작은 여기서 하면 된다.
    // subscriber.receive는 세 가지 종류가 있다. 나중에 알아보도록!
    @objc private func eventHandler() {
        _ = subscriber?.receive(control)
    }
}

/// UIControl 이벤트를 발신하는 Publisher.
struct UIControlPublisher<Control: UIControl>: Publisher {
    typealias Output = Control
    typealias Failure = Never

    let control: Control
    let controlEvents: UIControl.Event

    init(control: Control, events: UIControl.Event) {
        self.control = control
        self.controlEvents = events
    }
    
    func receive<S>(subscriber: S) where S : Subscriber, S.Failure == UIControlPublisher.Failure, S.Input == UIControlPublisher.Output {
        let subscription = UIControlSubscription(subscriber: subscriber, control: control, event: controlEvents)
        subscriber.receive(subscription: subscription)
    }
}

// 그리고 UIControl이 그 자체로 이벤트를 발신할 수 있도록 해 준다.
protocol CombineCompatible { }
extension UIControl: CombineCompatible { }

extension CombineCompatible where Self: UIControl {
    func publisher(for events: UIControl.Event) -> UIControlPublisher {
        return UIControlPublisher(control: self, events: events)
    }
}
```
- 이렇게 하면 위의 동작을 아래와 같이 바꿀 수 있다.
```Swift
final class MyPublishedViewController: MyViewController {
    override init() {
        super.init()
        
        // 상속받았으니까 중복되는 이벤트는 삭제하고
        incementButton.removeTarget(nil, action: nil, for: .touchUpInside)
        
        // 새로운 액션 세팅 함수 사용
        bindAction()
    }

    private func bindAction() {
        incrmenetButton.publisher(for: .touchUpInside)
            .sink { event in
            viewModel.incrementNumber()
        }
        .store(in: &cancellables)
    }
}
```