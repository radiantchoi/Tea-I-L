# Making custom SwiftUI dropdown menu
https://medium.com/stackademic/swiftui-dropdown-menu-3-ways-picker-menu-and-custom-from-scratch-7898d718c566

## 컨셉
- `Picker`, `Menu`로는 만족할 수 없는 당신을 위한 선택!
- 먼저 상호작용할 버튼이 필요하다.
- 버튼을 누르면 메뉴가 촤르륵 펼쳐진다.
- 이 펼쳐진 메뉴가 다른 화면의 위에 있어야 하므로, Z 우선순위가 높아야 한다.
- 메뉴를 "선택" 하면, 표시되는 메뉴가 바뀐다.
- 그러므로 드랍다운 메뉴를 여는 버튼과, 선택지로서 작용하는 버튼이 세로로 쌓여 있을 필요가 있다.
- 기왕이면 지금 선택되어 있는 아이템을 표시해주는 게 좋다.
- 드랍다운 메뉴가 너무 무한정 길면 안 되니까, 표시할 아이템의 갯수도 정해 줘야 하고..

## 실제
[직접 실행을 위한 앱 저장소 링크](https://github.com/radiantchoi/LikelyToUse/tree/main/SomethingUsefulSwiftUI/SomethingUsefulSwiftUI)
- 커스텀 드롭다운을 불러오는 화면
```Swift
import SwiftUI

struct CustomDropdownView: View {
    @State private var selectedOptionIndex: Int = 0
    @State private var showDropdown: Bool = false
    
    var body: some View {
        CustomizableDropdown(
            options: ["Robin", "Firefly", "Jade"],
            layoutOptions: CustomizableDropdownLayoutOptions(
                menuWidth: 300,
                buttonHeight: 100,
                maximumItemDisplayed: 2
            ),
            selectedOptionIndex: $selectedOptionIndex,
            showDropdown: $showDropdown
        )
    }
}
```
- 커스텀 드롭다운. 뷰의 크기, 최대 표시 가능 아이템의 수 등 많은 부분에서 커스터마이징이 가능하도록 약간 개조했다.
```Swift
import SwiftUI

struct CustomizableDropdownLayoutOptions {
    let menuWidth: CGFloat
    let buttonHeight: CGFloat
    let maximumItemDisplayed: Int
}

struct CustomizableDropdown: View {
    private let options: [String]
    private var menuWidth: CGFloat
    private var buttonHeight: CGFloat
    private var maximumItemDisplayed: Int
    
    @Binding var selectedOptionIndex: Int
    @Binding var showDropdown: Bool
    
    @State private var scrollPosition: Int?
    
    init(
        options: [String],
        layoutOptions: CustomizableDropdownLayoutOptions,
        selectedOptionIndex: Binding<Int>,
        showDropdown: Binding<Bool>
    ) {
        self.options = options
        self.menuWidth = layoutOptions.menuWidth
        self.buttonHeight = layoutOptions.buttonHeight
        self.maximumItemDisplayed = layoutOptions.maximumItemDisplayed
        self._selectedOptionIndex = selectedOptionIndex
        self._showDropdown = showDropdown
    }
    
    var body: some View {
        VStack {
            VStack(spacing: 0) {
                Button {
                    withAnimation {
                        showDropdown.toggle()
                    }
                } label: {
                    HStack(spacing: nil) {
                        Text(options[selectedOptionIndex])
                        
                        Spacer()
                        
                        Image(systemName: "chevron.down")
                            .rotationEffect(.degrees(showDropdown ? -180 : 0))
                    }
                }
                .padding(.horizontal, 20)
                .frame(width: menuWidth, height: buttonHeight, alignment: .leading)
                
                if showDropdown {
                    let scrollViewHeight: CGFloat = options.count > maximumItemDisplayed
                    ? buttonHeight * CGFloat(maximumItemDisplayed)
                    : buttonHeight * CGFloat(options.count)
                    
                    ScrollView {
                        ForEach(0..<options.count, id: \.self) { index in
                            Button {
                                withAnimation {
                                    selectedOptionIndex = index
                                    showDropdown.toggle()
                                }
                            } label: {
                                HStack {
                                    Text(options[index])
                                    
                                    Spacer()
                                    
                                    if index == selectedOptionIndex {
                                        Image(systemName: "checkmark.circle.fill")
                                    }
                                }
                            }
                            .padding(.horizontal, 20)
                            .frame(width: menuWidth, height: buttonHeight, alignment: .leading)
                        }
                        .scrollTargetLayout()
                    }
                    .scrollPosition(id: $scrollPosition)
                    .scrollDisabled(options.count < maximumItemDisplayed)
                    .frame(height: scrollViewHeight)
                    .onAppear {
                        scrollPosition = selectedOptionIndex
                    }
                }
            }
            .foregroundStyle(Color.white)
            .background(
                RoundedRectangle(cornerRadius: 15)
                    .fill(Color.black)
            )
        }
        .frame(width: menuWidth, height: buttonHeight, alignment: .top)
        .zIndex(100)
    }
}
```