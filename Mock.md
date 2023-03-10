# MOCK

## Mockito annotation을 활성화

### @RunWith(MockitoJUnitRunner.class)

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
	...
}
```

### MockitoAnnocations.openMocks()

- openMocks() 메소드를 사용하여 mockito annotation을 프로그래밍적으로 활성화할 수 있음

### MockitoJUnit.rule()

- rule() 메소드를 사용할 수 있음

```java
@Rule
public MockitoRule mockRule = MockitoJUnit.rule();
```

## @Mock

- Mock은 진짜 객체처럼 동작하지만 진짜 객체는 아닌 객체라고 할 수 있다.
- Mockito.mock을 부르기보다 @Mock 애노테이션을 사용하여 mock을 생성하거나 주입할 수 있다.

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
	// Mockito.mock() 메소드를 통해서 mock 리스트를 만들었다.
	List mockList = Mockito.mock(ArrayList.class);
}

@Mock
List<String> mockList;

@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
	mockList.add("one");
	...
}
```

## @Spy

- Spy는 진짜 오브젝트를 생성한다. 하지만 Mock과 달리 stub이 필요한 부분에만 stub을 할 수 있다.
- stub은 해당 메소드를 호출했을 때 반환해야하는 값을 미리 지정한다는 것을 의미한다.
- Mock과는 다르게 실제 오브젝트를 생성하므로, void값을 리턴한다면, mock은 stub하지 않는다면 return을 null, default value로 하는 반면, spy는 stub하지 않는다면 실제 메소드 동작을 호출하게 된다.

```java
@Spy
List<String> spiedList = new ArrayList<String>();

@Test
public void whenUseSpyAnnotation_thenSpyIsInjectedCorrectly() {
	spiedList.add("one");
  spiedList.add("two");

  Mockito.verify(spiedList).add("one");
  Mockito.verify(spiedList).add("two");

  assertEquals(2, spiedList.size());

  Mockito.doReturn(100).when(spiedList).size();
  assertEquals(100, spiedList.size());
}
```

- Mock과는 다르게 new ArrayList<String>() 을 통해 초기화해준다.
- spiedList.add() 는 실제로 리스트에 2개의 원소를 넣는다.
- doReturn(100)을 통해 실제로 spiedList.size()는 2가 아닌 100을 리턴하게 된다.

## @Captor

- ArgumentCaptor를 사용하여 특정 argument를 저장했다가 getValue()로 다시 사용할 수 있다.

```java
@Mock
List mockedList;

@Captor 
ArgumentCaptor argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSame() {
  mockedList.add("one");
  Mockito.verify(mockedList).add(argCaptor.capture());

  assertEquals("one", argCaptor.getValue());
}
```

## @InjectMocks

- 오브젝트의 field에 mock을 자동으로 주입시킨다.

```java
@Mock
Map<String, String> wordMap;

@InjectMocks
MyDictionary dic = new MyDictionary();

@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
  Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning");

  assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```

```java
public class MyDictionary {
    Map<String, String> wordMap;

    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```

- wordMap을 mock으로 만든다. wordMap은 MyDictionary라는 클래스의 필드이다.
- @InjectMocks 애노테이션은 자동으로 필드에 wordMap이라는 mock을 주입시킨다.

### Mock을 Spy에 주입시키는 법

- mockito는 Mock을 Spy에 주입시키는 방법은 없다.

```java
MyDictionary(Map<String, String> wordMap) {
    this.wordMap = wordMap;
}

@Mock
Map<String, String> wordMap; 

MyDictionary spyDic;

@Before
public void init() {
    MockitoAnnotations.openMocks(this);
    spyDic = Mockito.spy(new MyDictionary(wordMap));
}
```

- 따라서 생성자를 하나 만들고, Mockito.spy() 메소드를 사용해서 주입시키면 된다.

### NullPointerException

```java
public class MockitoAnnotationsUninitializedUnitTest {

    @Mock
    List<String> mockedList;

    @Test(expected = NullPointerException.class)
    public void whenMockitoAnnotationsUninitialized_thenNPEThrown() {
        Mockito.when(mockedList.size()).thenReturn(1);
    }
}
```

- 실제로 mock을 사용할 때, NPE를 만나는 경우가 생긴다. 이는 처음에 @Mock 애노테이션을 활성화하기 위한 처리를 하지 않았을 확률이 높으므로 @RunWith()와 같은 @Mock 애노테이션 활성화를 하도록 하자.
