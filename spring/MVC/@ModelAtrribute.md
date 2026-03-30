
### Step 1. Argument Resolver의 결정

`DispatcherServlet`이 컨트롤러의 메서드를 호출하기 전, 파라미터에 `@ModelAttribute`가 붙어있거나 생략되었더라도 단순 타입(String, int 등)이 아니면 `ModelAttributeMethodProcessor`가 해당 파라미터를 처리하기로 결정합니다. (`supportsParameter` 메서드 검증)

### Step 2. 객체 생성 (Instantiation)

가장 먼저 바인딩할 객체를 생성합니다.

- **우선순위:** 1. `@SessionAttributes`에 같은 이름의 객체가 있는지 확인. 2. 클래스 내부에 `@ModelAttribute`가 붙은 메서드가 있다면 해당 메서드를 미리 실행하여 반환된 객체를 사용. 3. 위 경우가 없다면 **기본 생성자(Default Constructor)**를 사용하여 객체를 생성합니다. (`BeanUtils.instantiateClass` 호출)
    

> [!warning] **주의 사항** 기본 생성자가 없다면 적절한 생성자를 찾으려 시도하지만, 매개변수가 없는 기본 생성자가 있어야 리플렉션을 통한 객체 생성이 가장 안정적입니다.

### Step 3. 데이터 바인딩 (Data Binding)

객체가 생성되면 HTTP 요청 파라미터(Query String 또는 Form Data)를 객체의 필드에 주입합니다. 이때 **`WebDataBinder`**가 사용됩니다.

- **동작 방식:**
    
    1. 요청 파라미터의 `name`과 객체의 `property` 이름을 비교합니다. (예: `?username=kim` -> `setUsername("kim")`)
        
    2. **Setter 메서드 호출:** 관례적으로 스프링은 Setter를 통해 값을 주입합니다. 만약 Setter가 없다면 값이 할당되지 않습니다.
        
    3. **PropertyEditor / ConversionService:** 요청 파라미터는 모두 문자열(String)입니다. 이를 객체의 필드 타입(`Integer`, `LocalDate` 등)에 맞게 변환하는 과정이 여기서 일어납니다.
        

### Step 4. 검증 (Validation)

파라미터 앞에 `@Valid` 또는 `@Validated`가 붙어 있다면, 바인딩 직후 **Validator**가 동작하여 객체의 제약 사항(예: `@NotNull`, `@Min`)을 검증합니다. 검증 결과는 `BindingResult` 객체에 담깁니다.

### Step 5. Model 추가 (Exposure)

바인딩이 완료된 객체는 지정된 이름으로 **`Model`**에 자동으로 추가됩니다.

- 별도의 이름을 지정하지 않으면 클래스 명의 첫 글자를 소문자로 바꾼 이름(예: `Member` -> `"member"`)으로 저장됩니다.
    
- 이 과정을 통해 별도의 `model.addAttribute()` 호출 없이도 뷰(JSP/Thymeleaf)에서 데이터를 바로 사용할 수 있습니다.