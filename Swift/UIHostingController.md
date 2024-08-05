# SwiftUI 컴포넌트를 UIKit 프로젝트에서 사용하기

## 발단
- UIKit 프로젝트에서 캡슐 모양으로 된 재사용 가능한 뷰를 만들려고 하는데, 생각보다 신경쓸 게 많고 레퍼런스도 적었음
- SwiftUI 컴포넌트 중 Capsule 컴포넌트를 활용해볼 수는 없을까?! 하는 생각을 하게 됨

## UIHostingController
- SwiftUI 뷰를 UIKit 프로젝트에서 사용할 수 있게 해 주는 도구
- 먼저 SwiftUI 뷰를 정의한다.
```Swift
import SwiftUI
import UIKit

// 레이블과 아이콘을 넣을 수 있고, 폰트와 테두리 그리고 내부 채우기 색상을 커스터마이징할 수 있는 캡슐 뷰
struct CapsuleView: View {
    let icon: UIImage?
    let title: String
    
    let backgroundColor: UIColor
    let contentsColor: UIColor
    let borderColor: UIColor
    
    let fontSize: CGFloat
    
    var body: some View {
        HStack(spacing: 5) {
            if let icon {
                Image(uiImage: icon)
                    .renderingMode(.template)
                    .frame(width: 8, height: 8)
            }
            
            Text(title)
                .font(Font(UIFont.notoSans(size: 12, family: .NotoSansKRMedium)))
        }
        .padding(.horizontal, 8)
        .padding(.vertical, 4)
        .foregroundColor(Color(uiColor: contentsColor))
        .background(
            Capsule().fill(
                Color(
                    uiColor: backgroundColor
                )
            )
            .overlay(
                Capsule()
                    .strokeBorder(Color(uiColor: borderColor), lineWidth: 1)
            )
        )
    }
}
```
- 다음으로 사용하고자 하는 뷰에서 `UIHostingController`를 통해 해당 뷰를 불러오고 메인 뷰에 장착한다.
```Swift
enum DonationStatus {
    case ongoing
    case ended
    
    // 진행 상태에 따른 뷰 커스터마이징
    var badgeView: CapsuleView {
        switch self {
        case .ongoing:
            return CapsuleView(
                icon: nil,
                title: "진행중",
                backgroundColor: .white,
                contentsColor: UIColor(resource: .commonBlue),
                borderColor: UIColor(resource: .commonBlue),
                fontSize: 12
            )
        case .ended:
            return CapsuleView(
                icon: nil,
                title: "기간종료",
                backgroundColor: .white,
                contentsColor: UIColor(resource: .commonMiddleRed),
                borderColor: UIColor(resource: .commonMiddleRed),
                fontSize: 12
            )
        }
    }
}

final class DonationCell {
    // 다른 컴포넌트

    // 진행중 배지
    private lazy var ongoingCapsuleBadge: UIView = {
        let vc = UIHostingController(
            rootView: DonationStatus.ongoing.badgeView
        )
        
        let capsule = vc.view!
        return capsule
    }()
    
    // addSubview 및 constraint 잡아주는 작업
}
```
- `UIHostingController`는 SwiftUI 뷰를 `view` 프로퍼티로 가지고 있는 하나의 뷰 컨트롤러.
- 여기서는 뷰만 사용할 것이기 때문에 `view` 프로퍼티만 빼와서 반환했다.
  - `UIHostingController` 자체를 원래 뷰 컨트롤러의 자식 뷰 컨트롤러로 더하고 처리하는 방법도 있다.

## 결과
- 캡슐 모양 배지가 잘 추가됐다. (추후 스크린샷 첨부 예정)
- 다만 생각보다 돌아가는 부분이 많았어서, SwiftUI 자체에 익숙한 게 아니라면 그냥 UIKit 기반 방법을 찾아서 만드는 게 나을지도
- 현재 회사 앱은 향후 SwiftUI로의 리뉴얼을 앞두고 있어 선제적으로 적용했다.

## 참조
- https://sarunw.com/posts/swiftui-view-as-uiview/