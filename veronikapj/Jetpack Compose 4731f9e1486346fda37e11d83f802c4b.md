# Jetpack Compose 사용하기

Created By: PilJu BAE
Last Edited: Jul 4, 2020 10:02 PM
Links: https://youtu.be/U5BwfqBpiWU
events: Sessions, android11

# Compose?

```kotlin
@Composable
fun ProductLabel(product: Product) {
		Text("${product.quentity}x ${product.name}")
}
```

@Composable라는 단일 주석을 가진 함수만 만들면 끝

새 위젯을 만들기 위해서 클래스를 확장하거나 생성자를 덮어쓸 필요가 없음. 

Composable 함수는 Composable 함수에서만 호출 가능

@Composable은 함수의 유형을 변경하고 올바른 컨텍스트에서만 코드화할 수 있다.

# Compose function paradigm

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled.png)

Composable은 데이터를 가져와 UI로 변환하는 함수일 뿐

UI는 데이터의 함수 : 데이터가 비즈니스 로직에서 함수까지 흘러가기를 원한다. 

코드를 쓰는 것이 더 간단할 뿐만 아니라 추론도 쉬워진다.

## ReComposable

값이 바뀔 때마다 논리를 재평가하고 필요에 따라 Tree 업데이트

요소를 제거하거나 숨길 필요가 없음

```kotlin
@Composable
fun ProductLabel(product: Product) {
		if ( product.quantity > 0 ) {
				Text("${product.quentity}x ${product.name}")
		}
}
```

UI를 업데이트하는 방식이 아니라 원하는 UI를 설정하는 방식

# Compose 작동 방식

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%201.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%201.png)

## Development Host

1. Kotlin 컴파일러

    [trailing lambdas](https://kotlinlang.org/docs/reference/lambdas.html#passing-a-lambda-to-the-last-parameter)

    주석을 사용하지만 주석 프로세서는 사용하지 않음

2. Composer 컴파일러 플러그인을 사용

    타이핑 프런트 시스템과 타입 시스템 레벨, 코드 생성 레벨에서도 작동

3. Android Studio

    내부에 Compose-특화된 도구들을 가지고 있음.

## Device

1. Compose Runtime

    Compose는 Android나 UI에 대해 전혀 몰라야 한다. 

    트리에서만 작동하기 때문에 UI들을 제외한 다른 것들은 Compose로 빼 버릴 수 있다.

2. Compose UI 코어

    입력 관리, 측정, 도면과 레이아웃

    행과 열과 같은 표준 레이아웃 및 기본 상호작용의 기초가 된다.

3. 머티리얼 디자인 Compose

# Jetpack Compose의 도구

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__7-31_screenshot.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__7-31_screenshot.png)

왼쪽 - 내장 에뮬레이터

오른쪽 - Jetpack Compose 의 미리보기

앱을 실행하지 않고도 위젯의 로직을 테스트할 수 있다.

# Modifiers

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%202.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%202.png)

개발자 프리뷰 1에 처음 소개 

Modifiers로 할 수 있는 것이 일반 컴포저블로도 할 수 있었음. (ex padding)

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__10-6_screenshot.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__10-6_screenshot.png)

# Lists

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%203.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%203.png)

```kotlin
@Composable
fun ShoppingCart(
		shoppingCart: LiveData<List<Product>> //RXJava or Flow도 지원 가능
) {
		val products by shoppingCart.**observeAsState(emptyList())**
		**AdapterList**(
					data = products
		) { product ->
				ShoppingCartItem(product) {
						Viewer3d(product)
				}
		}
}
```

# ConstraintLayout

## ContractLayout을 Compose와 쓰는 방법

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%204.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%204.png)

```kotlin
@Composable
fun ShoppingCartItemRow(/* ... */) {
		ConstraintLayout(
				constraintSet = ConstraintSet { // 완전히 코드로 정의 가능
						
						// tag가 제약조건을 개체로 반환하므로 다른 제약조건에서 참조로 사용할 수 있다.
						// 첫 번째 왼쪽 감소 단추의 constraint 지정
						val decreaseConstraint = tag(DecreaseTag).apply { // tag 지정
								// tag 안에 제약조건을 스스로 선언할 수 있음
								left constraintTo parent.left
								centerVertically()
						}

						// 버튼을 만들고 태그를 지정하면 constraint를 composable과 일치시킴
						// 오른쪽 다음 단추의 새로운 태그 생성
						val increaseConstraint = tag(IncreaseTag).apply {
								left constraintTo decreaseConstraint.right
								left.margin = 4.dp
								centerVertically()
						}
				}
		) {
			// ...
		}
}
```

## 약간의 로직을 추가할 수 있다면?

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%205.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%205.png)

```kotlin
@Composable
fun ShoppingCartItemRow(/* ... */) {
		ConstraintLayout(
				constraintSet = ConstraintSet { 
						// ...
						//일부 항목에서만 나타나는 라벨과 빨간색 스위치을 오른쪽으로 맞추려면?
						tag(AmoutTag).apply {
								// swatch 가 있으면 swatch의 constraint를 사용하고, 
								// 없다면 전 라벨의 constraint를 사용하겠다.
								left constraintTo (if (hasSwatch) colorConstraint 
																	 else labelConstraint).right
								horizontalBias = 1.0f
								centerVertically()
						}
				}
		) {
			// ...
			Text (
					modifier = Modifier.tag(AmountTag), 
					text = formatAmount(product)
			)
		}
}
```

xml 과 코드를 동시에 사용하는 것보다 훨씬 편리해진다.

# Animation

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%206.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%206.png)

```kotlin
// 쇼핑 카트 중 하나를 선택할 때마다 반지름 애니메이션 적용
// 모서리 항목 위에 오버레이 애니메이션화하고 체크표시를 애니메이션화 

// select 상태 추적 
val (selected, onSelected) = state { false }

// 다른 코너의 반지름을 애니메이션화 
val tlRadius = animate(if (selected) 48.dp else 8.dp)
val radius = animate(if (selected) 0.dp else 8.dp)

Surface(
		shape = RoundedCornerShape(
				topLeft = rlRadius,
				topRight = radius, bottomLeft = radius, bottomRight = radius
		)
) {
		// selected 상태를 값으로 부여하고 onSelected lambda 값이 바뀔때마다 호출
		Toggleable(
				value = selected,
				onValuedChange = onSelected,
				//ripple 효과
				modifier = Modifier.ripple()
		) {
				//...
		}

```

# Interoperability

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%207.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Untitled%207.png)

3D 렌더링 

## View inside Compose

```kotlin
//3d viewer 
@Composable
fun Viewer3d(product: Product) {
		var modelViewer by state<ModelViewer?> { null }
		
		// 원하는 UI 트리에 컴포저블이 처음 추가될 때 호출
		onActive {
				// Setup Choreographer callback
				// 화면을 새로 고칠 때마다 모든 프레임이 호출할 수 있도록 
				onDispose {
						// composable이 트리에서 사라지면 정리하는 곳
						// Remove Choreographer callback
						// adapterlist와 같은 경우 옳은 일을 하고 있는 지 확인할 수 있음
				}
		}

		onCommit(product) {
				// 데이터변경에 대응해서 색상을 변경함 
				// Setproduct color on 3D scene
		}

		// composable AndroidView function
		// xml 레이아웃을 위한 id 를 매개변수로 제공
		AndroidView(R.layout.surface_3d) { view -> 
				surfaceView = view as SurfaceView
				// 3d 애니메이션을 렌더링 하려면 서피스뷰를 새로 고쳐야 함
				modelViewer = setupModelViewer(surfaceView)
		}
}
```

다른 유형의 뷰(지도, 카메라)를 사용하려면 이러한 방식으로 Compose UI에 포함할 수 있다.

# Testing

Compose 안에서는 함수들에 적용하고 있기 때문에 위젯을 참조할 수 없고, ID의 개념이 없기 때문에 'findViewById'를 할 수 없다.

## Semantics

시멘틱은 컴포저블 트리에 메타데이터를 추가하는 방법

액세스 능력을 높이기 위해 사용, 테스트의 핵심

```kotlin
@Composable
fun ShoppingCartItem(
		product: Product,
		decrease: (product) -> Unit = { }
) {
		//...
		
		Row {
				// tag가 포함된 시멘틱 노드 생성
				// UI의 해당 부분을 식별하는 방법이고, 테스트하고 싶은 버튼만 포함시킨다.
				**TestTag(tag = Tags.ShoppingCartItemDecrease) {**
						SmallButton(onClick = decrease(product) }) {
								Image(Icons.Sharp.Remove)
						}
				}
				//...
		}
}
```

```kotlin
// 2개의 수량에서 사용자가 버튼을 클릭할 때마다 1개씩 수량을 줄이는 테스트
@Test
fun changeQuantity() {
		var product = Product(quentity = 2)
		val decrease: (Product) -> Unit = { 
				product = it.copy(quantity = it.quantity - 1) 
		}
		
		composeTestRule.setContent {
				ShoppingCartItem(
					product = product,
					decrease = decrease
				)
		}

		**findByTag(Tags.ShoppingCartItemDecrease)
				.doClick()
				.doClick()

		runOnIdleCompose {
				assertThat(product.quentity).isEqualTo(0)
		}**
}
```

![Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__23-27_screenshot.png](Jetpack%20Compose%204731f9e1486346fda37e11d83f802c4b/Jetpack_Compose__23-27_screenshot.png)