---
title: Spring Validation 다양한 처리 방법
catalog: true
date: 2018-12-04 01:03:21
subtitle:
header-img: "bg_computer.jpg"
tags:
- Framework
- Spring
catagories:
- Spring
- Guide

---


### org.springframework.validation 활용
~~~ java
@PostMapping(value = "/leave")
public String applyLeave(@ModelAttribute @Valid RequestDto dto,
                          Errors erros, RedirectAttributes redirectAttributes) {

    boolean isValid = validatorObject.validate(dto, erros);
    if (!isValid || erros.hasErrors()) {
        redirectAttributes.addFlashAttribute("alertMsg", erros);
        return "redirect:/member/v1/leave";
    }
    ...
}
~~~


~~~ java
public class ValidatorObject implements ControllerValidator {

    @Override
    public boolean validate(Object requestDTO, Errors errors) {
        // TODO
    }
}
~~~

**언제 사용 할까?**  
Controller단에서 @Valid에서 체크 할 수 없는 비지니스 유효성 검사를 처리하기 위해 사용한다.  


**장점은**  
실제 비지니스로직과 유효성 검사 체크하는 코드를 관심 분리하여 비지니스 로직에 집중 할 수 있다.  


**단점은**
비지니스로직과 유효성검사 로직이 분리 되어 있기 때문에 만약에 다른 곳에서 해당 기능을 이용하고자 할 때 유효성 검사 로직과 비지니스 로직을 같이 가져가야 하는 지식을 알고 있어야 쓸 수 있다. 또한 다른 개발자가 해당 기능을 사용하고자 할 때 비지니스로직만 호출하여 사용 할 경우 유효성 검사 체크 부분은 빠트릴수 있다.




### Custom Annotation 활용

~~~ java
@Documented
@Constraint(validatedBy = SortColumnTypeValidator.class)
@Target({ElementType.METHOD, ElementType.FIELD, PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface Target {

    // 애노테이션 지정 시 validatino rule에 맞는 메시지 지정 가능!!!
    String message() default "사용 예시) sort=title,asc or sort=title,name,desc or sort=title,desc&sort=name,asc";

    String[] names() default {};

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
    @Retention(RUNTIME)
    @Documented
    @interface List {
        NotBlank[] value();
    }
}

~~~

~~~ java
public class TargetValidator implements ConstraintValidator<Target, String> {

    @Override
    public void initialize(Target target) {
    
    }
 
    @Override
    public boolean isValid(String field, ConstraintValidatorContext cxt) {
        return field != null && field.matches("[0-9]+")
                && (field.length() > 8) && (field.length() < 14);
    }
}
~~~


### 비즈니스 로직에서 처리
제일 간단한 방법이다.  
그냥 Service Layer에서 유효성 검사 체크를 한다.  단, 유효성 검사에 대한 부분은 따로 메소드를 추출한다.


### Entity객체에서도 유효성 검사를 하자
