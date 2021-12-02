## 21년 12월 2일
- 오토레이 아웃 실수로 인한 삽질

## AutoLayout
- 제약 조건(Constraints)에 따라  뷰 계층 구조에 있는 모든 뷰의 크기와 위치를 동적으로 지정하는 것
- 핵심은 '자동으로' 컴포넌트의 위치를 동적으로 '지정'하는 것이다.
- 그렇기 때문에 위치 추론이 불가능하게 제약 조건을 설정하면 원하는대로 동작하지 않는다.

## Example
- 아래와 같이 safeArea 유/무에 따라 파란색 뷰가 보이고/안 보이는 뷰를 만든다고 할때

<img width="364" alt="스크린샷 2021-12-02 오후 8 20 39" src="https://user-images.githubusercontent.com/74946802/144412651-f83b22a2-bc47-4a91-96d0-e88189cbbbb6.png"> <img width="443" alt="스크린샷 2021-12-02 오후 8 19 12" src="https://user-images.githubusercontent.com/74946802/144412454-8557bfd6-7286-4634-adba-4cdda117f39b.png">

- 다음과 같이 코드를 작성했다.
```swift
import UIKit
import SnapKit
import Then
import RxSwift
import RxCocoa

final class TestView: UIView {
    
    private lazy var containerView = UIView().then {
        $0.backgroundColor = .red
    }
    
    private lazy var titleLabel = UILabel().then {
        $0.text = "Button"
        $0.textAlignment = .center
        $0.backgroundColor = .brown
    }
    
    fileprivate lazy var coverButton = UIButton()
    
    private lazy var safeAreaView = UIView().then {
        $0.backgroundColor = .blue
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        layout()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        fatalError()
    }
    
    private func layout() {
        
        addSubview(containerView)
        addSubview(safeAreaView)
        
        containerView.snp.makeConstraints {
            $0.leading.trailing.equalToSuperview()
            $0.bottom.equalTo(safeAreaLayoutGuide)
            $0.height.equalTo(60)
        }
        
        safeAreaView.snp.makeConstraints {
            $0.leading.trailing.bottom.equalToSuperview()
            $0.top.equalTo(containerView.snp.bottom)
        }
        
        containerView.addSubview(titleLabel)
        containerView.addSubview(coverButton)
        
        titleLabel.snp.makeConstraints {
            $0.leading.trailing.bottom.equalToSuperview().inset(10)
            $0.top.equalToSuperview()
        }
        
        coverButton.snp.makeConstraints {
            $0.edges.equalTo(titleLabel)
        }
    }
}

extension Reactive where Base: TestView {
    var tap: ControlEvent<Void> { base.coverButton.rx.tap }
}
```

```swift
import UIKit
import RxSwift

final class ViewController: UIViewController {

    private lazy var testView = TestView()
    private let bag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(testView)
        
        testView.snp.makeConstraints {
            $0.leading.trailing.bottom.equalToSuperview()
        }
        
        testView.rx.tap
            .subscribe(onNext: { tap in
                print(tap)
            }).disposed(by: bag)
    }
}

```

## Error
- 이 코드에는 아주 큰 문제가 있는데... 일단 아래 뷰 계층에서 볼 수 있듯이 TestView가 계층에 올라와 있지 않다.

<img width="163" alt="스크린샷 2021-12-02 오후 8 23 57" src="https://user-images.githubusercontent.com/74946802/144413125-66ab8f0c-6b08-4164-abc4-00b56405ef11.png">

- TestView의 subview들은 모두 올라와서 보기에는 정상적으로 보이나, superView인 TestView가 뷰 계층에서 확인되지 않는다.
- 원하던대로 보이면 문제가 없는 거 아니야? -> 라고 하기엔 button도 동작하지 않는다.

## 코드 수정
- 오늘의 주제에서 알 수 있듯이 AutoLayout을 잘 못 설정하면 이런 일을 겪게된다.
- 위 코드에서 잘못된 점은 무엇일까?
- 바로 containerView의 top을 설정해주지 않고, testView의 높이도 설정해주지 않았다는 점이다.
- testView를 그리기 위해서는 필수 제약 조건을 충족시켜줘야 하는데, Y축에 대한 정보가 모자라다는 것이 그 이유다.
- 그렇다고 testView.height를 설정한다면 safeArea에 따라 파란색 뷰를 없앨 때 문제가 생긴다.
- 그렇기 때문에 containerView.equalToSuperView()를 설정해주면 문제가 해결된다.

## 결과
- 뷰 계층에서 testView도 나타났고, 버튼도 잘 눌린다.

<img width="231" alt="스크린샷 2021-12-02 오후 8 29 06" src="https://user-images.githubusercontent.com/74946802/144413888-62124463-d600-4e65-b41c-914ca8ddf0a9.png">

