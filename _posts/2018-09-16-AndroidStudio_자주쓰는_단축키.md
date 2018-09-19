Android Studio 자주사용하는 단축키 모음
====
​
# General
1. Synchronize`Control + Alt + Y` : Gradle 파일 동기화 시켜주는 기능인듯하다.
2. Open settings dialogue`Control + Alt + S` : IDE Settings 창에 대한 단축키.
3. Open project structure dialog`Control + Alt + Shift + S` : 현재 Project에 대한 project structure창에 대한 단축키.
4. Switch between tabs and tool window`Control + Tab` : 이 기능도 참 유용한 것 같다. 각 메뉴에 대한 번호만 외우고 있다면 IDE에서 메뉴 전환 및 Open이 정말 편해진다.
​
# Navigating and Search
1. Search everything including code and menus`Press Shift twice` : 정말이지 유용한 기능이다. IDE가 인덱싱이 얼마나 잘되어 있는지 알 수 있다. 원하는 모든 것을 찾을 수 있고 Shift 키를 두번 누르면 되기 때문에 접근성도 뛰어나고 정말 강추하는 단축키.
2. Search by symbol name`Control + Alt + Shift + N` : Android에서 R로 생성되는 symbol만 검색하는 단축키.
3. Open file structure pop-up`Control + F12` : 현재 파일에 대한 Structure를 작은 팝업에 보여주는 단축키다. Structure 탭을 끄고 개발한다면 유용한 단축키.
4. Navigate between open editor tabs`Alt + Right/Left Arrow` : Editor창과는 무관하고 Project, Packages, Android 등의 탭 등을 빠르게 변환해주는 단축키.
5. Go to last edit location`Control + Shift + Backspace` : 마지막으로 코드를 수정했던 위치로 이동한다.
6. Hide active or last active tool window`Shift + ESC` :  활성화 되있거나 마지막에 사용한 tool window를 닫아주는 단축키.
7. Open type hierarchy`Control + H` : 선택된 class 등의 계층구조를 펼쳐서 보여주는 단축키.
8. Open method hierarchy`Control + Shift + H` : 선택된 method의 계층구조를 펼쳐서 보여주는 단축키. 소스 분석이나 흐름등을 파악할 때 유용하다.
9. Open call hierarchy`Control + Alt + H` : 선택된 method의 호출구조를 파악할때 유용하다. 어디서 호출되는지 펼쳐서 보여주는 단축키.
​
# Writing code
1. Generate code (getters, setters, constructors, hashCode/equals, toString, new file, new class) `Alt + Insert` : 오버라이딩 가능한 메소드가 무엇이 있는지 확인하거나 생성할 때 유용한 단축키.
2. Override methods`Control + O` : 1번과 비슷하지만 1번은 getters, setters 등 조금 더 범위가 넓다.
3. Quick documentation lookup`Control + Q` : 주석등을 확인할 때 유용하다. 기존 Eclipse에서는 커서를 갖다 대면 주석등을 확인할 수 있었는데 Android Studio는 그런 기능이 없다. 대신 이 단축키를 사용하면 편리하고 접근성도 좋다.
4. Show parameters for selected method`Control + P` : 선택된 메서드의 파라미터 목록을 툴팁형태로 제공하는 단축키.
5. Open quick definition lookup`Control + Shift + I` : 선택된 영역의 메서드나 클래스의 코드 구현 내용을 팝업 형태로 제공하는 단축키.
6. Toggle project tool window visibility`Alt + 1` : Project tool window를 보여주고 감추고 토글 시켜주는 단축키.
7. Toggle bookmark`F11` : 특정 코드 라인에 북마크를 설정해두면 Shift + F11 키를 통해 다른 코드를 보다가 북마크해둔 코드 영역을 팝업 형태로 볼 수 있다.
8. Toggle bookmark with mnemonic`Control + F11` : 19번과 동일하지만 단순 북마크가 아니라 숫자나 영어 알파벳 등을 지정할수 있다.
9. Delete to end of word`Control + Delete` : word단위로 코드를 삭제해주는 기능인데 말로 설명하기 복잡하다. 한번 써보면 바로 이해가 될듯.
10. Optimize imports`Control + Alt + O` : import는 하였으나 사용하지 않는 클래스가 있을 경우 해당 import구문을 제거해준다.
11. Next/previous highlighted error`F2 / Shift + F2` : 보통 Lint에러나 컴파일 에러등 IDE에서 하이라이팅으로 처리되는 부분이 있는데 그부분으로 바로 이동하는 단축키다.
​
# Refactoring
1. Move`F6` : 보통 리팩토링할 때의 패키지 변경등 클래스의 위치를 변경할 때 사용하는 단축키.
2. Safe delete`Alt + Delete` : 삭제를 할 때 삭제를 하므로써 발생하는 다른 문제 등에 대해서 팝업으로 우선 알려주는 단축키.
3. Rename`Shift + F6` : 리팩토링할 때 제일 자주 사용되는 단축키다. 이름 변경해주는 단축키.
4. Extract method`Control + Alt + M` : 해당 코드 블럭을 메서드로 추출하기 위한 단축키.
5. Extract variable`Control + Alt + V` : 멤버 변수를 지역 변수로 변경하거나 할때 쓰는 단축키다.
6. Commit project to VCS`Alt + Control + A` : add 의 단축키.
7. Commit project to VCS`Control + K` : commit의 단축키.
8. View recent changes`Alt + Shift + C` : 최근 local 변경 사항을 확인할 수 있는 팝업을 제공하는 단축키.
