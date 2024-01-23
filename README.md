## Compose 수정자 "Modifier"
<br>

### 1. 수정자의 순서는 중요하다.

수정자의 각 함수는 이전 함수에서 반환한 `Modifier`를 변경하므로 순서는 최종 결과에 영향을 준다.   
즉 밑의 두 코드의 결과는 다르다.
```kotlin
//1
Modifier.clickable(...)
  .padding(...)

//2
Modifier.padding(...)
  .clickable(...)
```
<br>

### 2. Compose의 범위 안전성

**`matchParentSize()`는** 특정 컴포저블의 하위 요소 크기를 그 컴포저블 크기만큼 크게 만들기 위해 사용된다.
```kotlin
@Composable
fun MatchParentSizeComposable() {
    Box {
        Spacer(Modifier.matchParentSize().background(Color.LightGray))
        ArtistCard()
    }
}
```
위 코드의 경우 `Box`의 크기는 `ArtistCard()`에 의해 결정된다. `Spacer`의 크기가 `Modifier.matchParentSize()`로 결정되었기 때문에 그 크기가 
`Box`와 같아지고, 결과적으로 `ArtistCard()`와 같아지게 된다.
<br>

**`weight(float)`는** `RowScope` 및 `ColomnScope`에서 사용할 수 있는 가중치 형식 크기 지정 메서드다.
```kotlin
@Composable
fun ArtistCard(/*...*/) {
    Row(modifier = Modifier.fillMaxWidth()) {
        Image(modifier = Modifier.weight(2f))
        Image(modifier = Modifier.weight(2f))
    }
}
```
<br>

### 3. 수정자 추출 및 재사용 권고

- 자주 변경되는 상태 관찰을 위해 수정자 추출

  리컴포지션이 일어날 때마다 모든 컴포저블에 수정자가 재할당된다. 의 경우, 
  그만큼 수정자 재할당이 많이 일어난다. 따라서 리컴포지션이 많이 발생하는 스크롤이나 애니메이션 컴포저블의 경우 수정자를 따로 객체화하여 관리하는 편이 좋다.
  ```kotlin
  // Now, the allocation of the modifier happens here:
  val reusableModifier = Modifier
            .padding(12.dp)
            .background(Color.Gray)

  @Composable
  fun LoadingWheelAnimation() {
    val animatedState = animateFloatAsState(...)

    LoadingWheel(
        // No allocation, as we're just reusing the same instance
        // If modifier is declared here, creation and allocation will happen on every frame of the animation!
        modifier = reusableModifier,
        animatedState = animatedState.value
    )
  }
  ```

- **범위에 따른 수정자 추출**

  범위가 지정되지 않은 수정자는 변수로 추출하기 쉽다. 이런 수정자는 **Lazy 레이아웃**과 사용하기 유용하다.

  범위가 지정된 수정자는 가능한 높은 수준의 컴포저블로 수정자를 추출하고 적절한 경우 재사용하는 방법을 사용한다.   
  이렇게 추출된 수정자는 동일한 범위의 직접 하위 요소에만 전달해야 한다.

- **추출된 수정자 추가 체이닝**
  
  **`.then()`함수**를 통해 다른 수정자에 추출한 수정자를 추가할 수 있다. 이때 수정자 함수의 순서가 중요하단 것을 까먹지 말자!
  ```kotlin
  otherModifier.then(reusableModifier)
  ```



