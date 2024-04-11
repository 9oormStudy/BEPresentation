## REST API 란 

<img width="573" alt="스크린샷 2024-04-11 오후 4 05 57" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/b522aa76-d04f-47d1-a4aa-7a8c9d61375d">

- REST API 란? 서버의 자원을 클라이언트에 구애받지 않고 사용할 수 있게 하는 설계 방식
  
### REST
- REST : 1. 자원을 이름으로 구분하여 2. 해당 자원의 상태를 주고받는 모든 것
- `HTTP URI` 를 통해 자원을 명시하고
- `HTTP Method`(GET, PUT, DELETE, PATCH) 를 통해 자원에 대한 CRUD (생성, 조회, 수정, 삭제) 를 적용

<img width="746" alt="스크린샷 2024-04-11 오후 4 40 01" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/099e568f-b481-4e71-bac4-a91b6ea85a27">

1. 자원을 이름으로 구분? = 자원의 표현
   - 자원을 HTTP URI 를 통해 명시
   - EX) 영화 정보가 자원일 때, /movie를 자원의 표현으로 정한다.
2. 자원의 상태를 주고 받음 (요청 -> 응답)
   - 클라이언트가 자원의 상태에 대한 조작을 요청하면 서버는 이에 적절한 응답을 보낸다.
   - JSON 혹은 XML 을 통해 데이터를 주고 받는 것이 일반적
  
즉, REST 는 HTTP URI 를 통해 Client 와 Server 사이의 통신하는 방식 중 하나로, HTTP Method 를 통해 자원을 처리 하도록 설계된 아키텍쳐를 말한다.   

### API
- API : 클라이언트가 서버의 자원을 요청할 수 있도록 서버에서 제공하는 인터페이스

### REST API
- REST API 란 결국 REST 를 기반으로 구현한 API

### RESTful API
<img width="517" alt="스크린샷 2024-04-11 오후 5 08 56" src="https://github.com/9oormStudy/BEPresentation/assets/53373279/3fe80cd3-3d87-4191-bfd2-2f78e0d217e1">

- RESTful 하다는 것은 REST 를 기반으로 잘 설계된 API 를 의미한다.
- RESTful 한 API 는 사진처럼 딱 봐도 어떠한 요청인지 한눈에 이해할 수 있다.

``` java

    //생성
    @PostMapping("/api/coffees")
    public ResponseEntity<Coffee> save(@RequestBody CoffeeDto dto) {
        Coffee saved = coffeeService.save(dto);
        return (saved != null) ?
                ResponseEntity.status(HttpStatus.OK).body(saved) :
                ResponseEntity.status(HttpStatus.BAD_REQUEST).build();
    }

    //수정
    @PatchMapping("/api/coffees/{id}")
    public ResponseEntity<Coffee> patch(@PathVariable Long id, @RequestBody CoffeeDto dto) {
        Coffee patched = coffeeService.patch(id, dto);

        return (patched != null)?
                ResponseEntity.status(HttpStatus.OK).body(patched) :
                ResponseEntity.status(HttpStatus.BAD_REQUEST).build();
    }
```
