# CEOS 17기 백엔드 스터디  

## [2주차]  

### ORM 이용해보기  
1. 게시글 생성  
![image](https://user-images.githubusercontent.com/90256209/229326431-a573d2b4-03b6-4c8c-a2bf-7e14ad507831.png)
2. 게시글 수정 및 쿼리셋 조회  
![image](https://user-images.githubusercontent.com/90256209/229326491-4cb57511-12ba-45a1-9274-ba3bde83da88.png)
3. filter 함수 사용  
filter의 `__contains`와 `__startswith`을 사용해봤다.
![image](https://user-images.githubusercontent.com/90256209/229326506-cf1c83a1-0b5c-43bd-bda9-5308a030df27.png)

---
### ERD 설계  
실무에서 ERD를 그린 후에 SQL문을 export해서 DB에 넣는 경우, ERD에 관계선으로 연관관계(Ex 1:N)를 설정해두면 테스트 데이터를 넣기가 불편해서 거의 하지 않는다고 배워서 습관적으로 관계선 설정을 안했더니 테이블간 관계가 한눈에 안들어오는것 같다  

![CEOS  everytime](https://user-images.githubusercontent.com/90256209/229327752-7a75378c-b28a-4298-96c8-baa159f407f6.png)
ERD를 설계하기전에 연관관계 매핑에 대해 생각해봤다.  
- User1과 User2의 관계 : User1은 여러명의 유저와 친구가 될 수 있고, User2도 여러명의 유저와 친구가 될 수 있으므로 다대다(N:M) 관계임 -> 중간 테이블 'Friend'로 관리  
- Board(게시판), Post(게시글), Comment(댓글)의 관계 : 각 게시판에 여러개의 글이 있고, 각 글은 하나의 게시판에 소속되므로 게시판과 게시글은 1:N 관계임. 각 게시글에 여러개의 댓글이 있을 수 있고, 각 댓글은 하나의 게시글에 소속되므로 게시글과 댓글도 1:N 관계  
- User가 Post/Comment를 작성하는 관계 : 유저는 여러개의 게시글/댓글을 쓸 수 있고, 각 게시글/댓글은 한명의 유저에 의해 쓰였으로 유저가 게시글/댓글을 작성하는 관계는 1:N 관계  
- User가 Post를 Likes(공감)과 Scrap(스크랩)하는 관계 : 유저는 여러개의 게시글을 공감/스크랩할 수 있고, 각 게시글은 여러명의 유저에 의해 공감/ 스크랩될 수 있으므로 다대다(N:M) 관계임 -> 중간 테이블 'Likes', 'Scrap'으로 관리  
- Photo(사진)와 Post의 관계: 각 게시글에 여러 사진을 올릴 수 있으므로 1:N
- TimeTable(시간표), Lecture(강의)의 관계: 각 시간표에 여러 강의를 추가할 수 있고, 각 강의도 여러 시간표에 포함될 수 있으므로 N:M -> 중간 테이블 'TakeLecture(수강)'로 관리
- LectureReview(강의평)와 Lecture의 관계: 각 강의에 여러 강의평이 달릴 수 있고, 각 강의평은 하나의 강의에 소속되므로 1:N 관계  

그리고 고민됐던건 시간표에 강의를 추가할때 '일반교양' > '16이후, 공학' > '인문계열' 이런식으로 전공/영역을 선택하면 강의가 여러개 나오는 부분이었다. 이렇게 전공/영역이 계층구조를 가지기 때문에 LectureDomain(전공/영역) 테이블을 따로 추가했다.  
예를들어,  
![image](https://user-images.githubusercontent.com/90256209/229328515-3fa2a18f-efb0-4c1c-b6b3-b38727d8a8a1.png)  
'한국근현대사' 강의가 lectureDomain_id로 3을 가지면, 인문계열 강의라는 뜻이고, parent_id를 통해 거슬러 올라갈 수 있다!  
parent_id의 default는 0으로 최상위 영역이다.  
강의 시간에 대해서도 고민이 됐는데 강의 테이블에 string으로 넣었다.

---  
### 겪은 오류와 해결 과정
처음에 model을 작성할때 굳이 역참조 안해도 되는 테이블들에 related_name을 다 넣었더니 Reverse accessor for ~ clashes with reverse accessor for ~ 에러가 났다. 구글링해보니 related_name이 필수인 경우는 한 테이블(클래스)에서 서로 다른 두 컬럼(필드)이 같은 테이블(클래스)을 참조하는 경우뿐이어서 나의 경우에는 해당되지 않아 삭제했다.

오류는 안났지만 User를 OneToOne 방식으로 확장할때 User에서 email, password, username과 같은 필드를 기본 제공해준다는걸 까먹고 Profile에 중복되는 필드를 넣어서 나중에 삭제했다.

---
### 새롭게 배운 점  
ManyToMany(다대다)관계를 설정하는 방법에 대해 알게되었다. 다대다 관계에서 중간테이블을 만든다는건 알고있었지만 Django에서 어떻게 모델링하는지(through를 통해서)는 처음 알게됐다. through는 반드시 다대다 관계중에서 한곳에만 선언해야 한다.  다대다 관련해서는 더 공부해야겠다,,
[참고1](https://velog.io/@jiffydev/Django-9.-ManyToManyField-1) [참고2](https://hoorooroob.tistory.com/entry/%ED%95%B4%EC%84%A4%EA%B3%BC-%ED%95%A8%EA%BB%98-%EC%9D%BD%EB%8A%94-Django-%EB%AC%B8%EC%84%9C-Models-%EB%8B%A4%EB%8C%80%EB%8B%A4-%EA%B4%80%EA%B3%84%EC%97%90%EC%84%9C%EC%9D%98-%EC%B6%94%EA%B0%80-%ED%95%84%EB%93%9C)  

---
### 느낀점  
장고를 쓰다보니 편한점들이 보이는것 같다. 일단 admin.py 기능은 최고야,, User 클래스에서 이미 많은 필드가 정의되어있는것도 신기했다.  
ERD 설계는 언제해도 고민되는 부분이 많은것 같다. 하다보니까 진짜 에브리타임 DB는 어떻게 되어있을지 궁금해졌다. 테이블이 엄청엄청 많겠지?  
모델을 분리했어야 하는데 이번 과제는 다른일들때문에 너무 늦게시작해서 아쉬움이 남는다 담부턴 더 미리 시작하자  

---
## [3주차]  

### API 명세
우선적으로 어떤 CBV API를 만들지 에브리타임을 구경하면서 리스트업해봤다
![image](https://user-images.githubusercontent.com/90256209/231825029-5d62ead3-c302-45c6-82ed-8cea6ec22733.png)  
게시글 삭제 및 수정 API에 대해 말을 하자면, 원래 DELETE 는 사용을 지양해야 한다고 들은지라… (사실 손사래 치시면서 절대 쓰지 말라고 하셨따 하하) 항상 PATCH 로 모델의 status 를 ‘D’로 바꾸는 식으로 개발했었는데 장고에서는 아무리 구글링해봐도 views.py 에서 수정을 이런식으로 구현한게 없더라 그래서 partial 을 활용해서 PUT 으로 status 필드만 수정할 수 있도록 구현했다(그냥 PUT 은 모든 필드를 다 채워서 요청해야해서 불편하니까) 찾아보니까 장고에서는 소프트 딜리트를 하기 위해서 SoftDelete 모델을 구현하는 것 같더라. 어쩐지! 나중에 시간 나면 해봐야겠다  

---
### 과제 진행 과정  
1.  패키지 수정  
    
    이전에 api 앱 속 [models.py](http://models.py) 안에 모든 모델이 있던 구조를 여러개의 앱으로 분리했다.  
    
2.  migration 초기화 
    
    모델 구조를 바꿔서 혹시나 하고 migration 파일들을 [init.py](http://init.py) 빼고 다 삭제했더니 아예 데이터베이스 자체가 삭제되서 에러가 나길래 `create database ceosDB;` 로 다시 만들어줬다.
    
3.  account / community / timetable 3개의 앱으로 분리하고, BaseModel 정의를 위한 utils 앱도 만들었다.  
4. API 명세서 작성 - 이번에는 게시글 관련 API들만 만들어봤다.
5. [serializers.py](http://serializers.py) 작성 
    
    게시글(Post)을 가져올때 글쓴이(Profile)의 __PK__, __이름__, __프로필사진URL__ 만 가져오도록 했다. (항상 API의 응답으로 PK는 주는게 좋다, 잊지말자) 
6. [views.py](http://views.py) 및 [urls.py](http://urls.py) 작성
7. 리팩토링 - 피드백 반영 및 Comment 테이블과 LectureDomain 테이블의 parent 컬럼 null 허용  
8. CBV API 작성
9. ViewSet으로 리팩토링 (짱신기)
10. Filter 기능 구현  

---
### 웹 브라우저와 Postman을 통한 API 테스트  

1. 게시글 생성 API (Method: `POST`, URL: `/community/boards/1/`)  
게시판을 지정해서 게시글을 작성해보자  
![image](https://user-images.githubusercontent.com/90256209/231829358-b2af2e20-e2d9-4516-98c6-d8ad87a7ad62.png)  

2. 게시글 수정 및 삭제 API (Method: `PUT`, URL: `/community/posts/1/`)  
partial update로 원래는 `'A'`였던 `status`를 `'D'`로 바꿔줌으로써 게시글을 삭제해보자  
물론 당연히 `title`이나 `contents` 수정도 가능하다  
![image](https://user-images.githubusercontent.com/90256209/231832804-54d1b27c-28b4-4ffe-9acd-1d7c2a573b36.png)  

3. 특정 게시글 조회 API (Method: `GET`, URL: `/community/posts/1/`)  
특정 게시글을 조회해보자  
방금 삭제했기 때문에 `status`가 `'D'`로 바뀐걸 볼수있다  
![image](https://user-images.githubusercontent.com/90256209/231834896-9d1bd72b-97c9-4c21-915c-f6a42942c3de.png)  

4. 전체 게시판 조회 API (Method: `GET`, URL: `/community/boards/`)  
일단은 PK랑 이름만 나오게 했다  
![image](https://user-images.githubusercontent.com/90256209/231837072-5a93ce5f-d33d-44c4-8ec4-2ce1705bb2ad.png)  

5. 특정 게시판 전체 게시글 조회 API (Method: `GET`, URL: `/community/boards/1/`)  
전체 게시글이 조회될때 삭제된 게시글은 보이면 안되니까 `status`가 `'A'`인것만 보이도록 _filter_ 해줬다  
원래는 있었는데요  
![image](https://user-images.githubusercontent.com/90256209/231837919-4b764e4a-5e49-4d6c-84ec-b2025317696f.png)  
삭제하고나면 없어요  
![image](https://user-images.githubusercontent.com/90256209/231838078-43ede8d3-be4b-440f-861f-acc41be05194.png)  

---
### ViewSet으로 리팩토링하기  
요건.. 너무 신기했다 이게 될까? 했는데 진짜 되더랑  
몇줄 안되는 코드로 이렇게 특정 게시글 디테일 보기 및 수정/삭제가 된다  
![image](https://user-images.githubusercontent.com/90256209/231839566-90d6656f-6908-4f44-80c5-c779ddba986e.png)  

---
### Filter 기능 구현하기  
Post가 외래키로 가지는 Board랑 Profile을 서로 다른 방식으로 filter해봤는데 둘다 잘된다.  
Board는  
```
board = filters.NumberFilter(method='filter_board')
# ...
def filter_board(self, queryset, board_id, value):
  return queryset.filter(**{
    board_id: value,
  })
```  
이렇게 메소드로 필터링해봤다.  
Profile은 `profile = filters.NumberFilter(field_name='profile_id')` 이렇게 NumberFilter로 필터링해줬다.  
Title은 `title = filters.CharFilter(field_name='title', lookup_expr='icontains')` 이렇게 'icontains'를 사용해서 검색할 수 있더라  
1. 게시판으로 필터링  
![image](https://user-images.githubusercontent.com/90256209/231843912-bf6032f7-a2bb-4e2a-996d-fd2558233c40.png)  
2. 제목으로 필터링  
![image](https://user-images.githubusercontent.com/90256209/231844302-22024d96-df75-4ac3-bb01-c5ee0eec933f.png)  
3. 글쓴이로 필터링  
![image](https://user-images.githubusercontent.com/90256209/231844565-8c26fa84-7f3f-4c2f-baa4-d96baf1aa42c.png)  

---
### 겪은 오류와 해결 과정  
- 앱을 분리하면서 `Field defines a relation with model 'Profile', which is either not installed, or is abstract` 에러가 나서, `models.ForeignKey("account.Profile", on_delete=models.CASCADE)` 이렇게 외래키에 app이름을 명시함으로써 해결했다.
- API 명세를 고민하다가 Profile 클래스에 `school_id` 를 외래키로 안 넣은걸 발견해서 수정했다. NOT NULL로 설정되어 있다보니 migration할 때 에러가 났는데, 이번만 default를 설정하는 옵션이 있어서 그렇게 해결했다.  
- 이후의 오류와 해결 과정은.. 너무 많은데 머리 싸매느라 못적었다  

---
### 느낀점  
- 장고에서는 어노테이션을 ‘데코레이터’라고 하던데 이름이 뭔가 귀엽다ㅎㅎ  
- 장고는 기본 제공해주는 기능들도 많지만 그만큼 custom할 수 있는 요소도 꽤 많은 것 같아서 생각보다 좋당 근데 구글링을 열심히 해도 자료가 좀 부족한 느낌이 있다ㅠㅠ 장고도 글 많이 써주세요  
- 핫게시판의 기준을 오로지 댓글수+공감수로 한다면 filter 로 구현할 수 있을 것 같다  
- 어짜피 실제로 사용할 만한 세분화된 기능들을 개발하려면 ViewSet안에서도 따로 정의해야할게 많은 것 같은데 이럼 CBV보다 더 좋은건지는 아직 잘 모르겠따  
- 다들 중간고사 잘 마무리하고 만나요👻  

---
## [5주차 - Simple JWT]  

## 👀 로그인 인증은 어떻게 하나요? JWT 는 무엇인가요?
### 1. Cookie & Session 기반 인증  
- Cookie: 클라이언트가 어떠한 웹사이트를 방문할 경우, 그 사이트가 사용하고 있는 서버를 통해 `클라이언트의 브라우저`에 설치되는 작은 기록 정보 파일  
- Session: 세션은 비밀번호 등 클라이언트의 인증 정보를 쿠키가 아닌 `서버 측에 저장하고 관리`, 브라우저 종료할 때까지 인증상태가 유지됨  
- 동작 방식:  
 1️⃣ 서버는 클라이언트의 로그인 요청에 대한 응답을 작성할 때, 인증 정보는 서버에 저장하고 클라이언트 식별자인 SESSION ID를 쿠키에 담음  
 2️⃣ 이후 클라이언트는 요청을 보낼 때마다, SESSION ID 쿠키를 함께 보냄  
 3️⃣ 서버는 SESSION ID 유효성을 판별해 클라이언트를 식별함  
- 장점: 각 사용자마다 고유한 세션 ID가 발급되기 때문에, 요청이 들어올 때마다 회원정보를 확인할 필요가 없음  
- 단점: 쿠키를 해커가 중간에 탈취하여 클라이언트인척 위장할 수 있는 위험성 존재, 서버에서 세션 저장소를 사용하므로 요청이 많아지면 서버에 부하가 심해짐  

### 2. JWT 기반 인증  
- JWT(JSON Web Token): 인증에 필요한 정보들을 암호화시킨 토큰  
- JWT 구조: `Header` , `Payload` , `Signature` 로 이루어짐. Header는 정보를 암호화할 해싱 알고리즘 및 토큰의 타입을 지정, Payload는 실제 정보(클라이언트의 고유 ID 값 및 유효 기간 등)를 지님, Signature는 인코딩된 Header와 Payload를 더한 뒤 비밀키로 해싱하여 생성 → 토큰의 위변조 여부를 확인하는데 사용됨
- 동작 방식:  
 1️⃣ 클라이언트 로그인 요청이 들어오면, 서버는 검증 후 클라이언트 고유 ID 등의 정보를 Payload에 담음  
 2️⃣ 암호화할 비밀키를 사용해 Access Token(JWT)을 발급함  
 3️⃣ 클라이언트는 전달받은 토큰을 저장해두고, 서버에 요청할 때 마다 토큰을 요청 헤더 Authorization에 포함시켜 함께 전달함  
 4️⃣ 서버는 토큰의 Signature를 비밀키로 복호화한 다음, 위변조 여부 및 유효 기간 등을 확인함  
 5️⃣ 유효한 토큰이라면 요청에 응답함  
 - 장점: 인증 정보에 대한 별도의 저장소가 필요없음, 확장성이 우수함  
 - 단점: 토큰의 길이가 길어, 인증 요청이 많아질수록 네트워크 부하가 심해짐  

### 3. OAuth 2.0을 이용한 인증  
- OAuth: 구글, 페이스북, 트위터와 같은 다양한 플랫폼의 특정한 사용자 데이터에 접근하기 위해 클라이언트(우리의 서비스)가 사용자의 접근 권한을 위임(Delegated Authorization)받을 수 있는 표준 프로토콜.  
쉽게 말하자면, 우리의 서비스가 우리 서비스를 이용하는 유저의 타사 플랫폼 정보에 접근하기 위해서 권한을 타사 플랫폼으로부터 위임 받는 것  
- 나의 앱은 클라이언트👧 / 사용자는 리소스 오너🙋‍♂️ / 구글, 카카오 같은 큰 서비스는 리소스 서버🧝 (사실 데이터 처리를 담당하는 Resource 서버와 인증을 담당하는 Authorization Server로 구성됨)  
- 동작 방식:  
![image](https://user-images.githubusercontent.com/90256209/236613747-47da422f-971f-4d1b-a4c2-2ed0f5dbd972.png)  

---
## 🗣️ 피드백 반영 및 수정사항  
1. 찬혁오빠가 구현한 ***safe delete*** 참고해서 base model에 delete 메소드 추가함  
   데이터 삭제시 deleted_at 컬럼에 삭제시간을 저장하는 방식 = deleted_at이 null이면 정상(=삭제 안된) 게시물  
2. "해당 글의 status는 D인데 조회했을때 보이면 안될 것 같아요🥹🥹"  
   → Filter 클래스에 ***deleted_at이 null인 것만*** 조건을 추가한 ***filter 메소드***를 구현함  

#### (수정한 filter 메소드)  
![image](https://user-images.githubusercontent.com/90256209/236614033-d4ef3388-8987-4dfb-bce0-fcd28f63b5a2.png)  
#### (결과 확인)  
⬇️ 이렇게 DB에서 `deleted_at` 컬럼에 삭제시간이 들어가 있는 경우,  
![image](https://user-images.githubusercontent.com/90256209/236614073-0a8baead-feec-4088-906c-55b99d86fed4.png)  
➡️ postman으로 조회했을때 안보이는걸 확인할 수 있다!! profile_id가 2인 글은 삭제되었으므로 필터링해도 안보임!  
![image](https://user-images.githubusercontent.com/90256209/236614174-b11ebd10-677d-424d-b7d0-5d542ce06e4d.png)  

---
## 👩‍💻 JWT 로그인 구현하기  

### 📌 커스텀 User 모델 사용하기  
`AbstractBaseUser` 를 상속한 커스텀 User 모델을 만들었다. (기존에는 기본 User 모델을 OneToOne 필드로 사용한 Profile 모델을 사용했었음)  
AbstractUser와 AbstractBaseUser의 차이는 기본 제공하는 필드들이 다르다! (AbstractUser가 더 많이 제공함ㅎㅎ)  
+유저 모델을 커스텀할 때
- `USERNAME_FIELD` 은 유저를 고유하게 식별할때 쓰는 필드고,
- `REQUIRED_FIELDS` 는 반드시 필요한 필드다.  
나는 유저 식별을 `email`로 하게끔 만들었다.  
```  
class MyUser(AbstractBaseUser):
    email = models.EmailField(max_length=255, unique=True)
    nickname = models.CharField(max_length=100, unique=True)
    # password, last_login 은 기본 제공
    profile_img_path = models.URLField(blank=True, null=True)
    friends = models.ManyToManyField('self', blank=True)
    school = models.ForeignKey("School", on_delete=models.CASCADE)
    is_admin = models.BooleanField(default=False)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    objects = MyUserManager()

    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = ["nickname"]

    def __str__(self):
        return self.email
```  
UserManager 클래스도 `BaseUserManager`를 상속받아서 커스텀해주었다.

```  
class MyUserManager(BaseUserManager):
    def create_user(self, email, nickname, school=None, password=None, **extra_fields):
        if not email:
            raise ValueError("Users must have an email address")

        user = self.model(
            email=email,
            nickname=nickname,
            school=school,
        )
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, nickname, school=None, password=None, **extra_fields):
```  
`create_superuser()`는 `create_user`와 거의 비슷하지만, `superuser.is_admin = True`를 자동 설정한다는 점이 다르다.  


### 📌 회원가입 구현하기  
Postman으로 확인해보니 잘된다ㅎㅎ  
![image](https://user-images.githubusercontent.com/90256209/236615823-7dde4a0b-a8f7-4824-aa42-f054c8362583.png)  
번외) 원래 회원가입할땐 자동로그인이 아니면 토큰 발급을 안한다. 근데 사진에서 쿠키에 뭔가가 있는건 직전에 테스트하던 회원 로그아웃을 안해서 아직 쿠키가 남아있음...ㅎ ~~(NG)~~  


### 📌 JWT Login 구현하기 (Access 토큰, Refresh 토큰 발급)  
로그인 구현할 때 **Access 토큰**은 **HTTP Response**로 프론트한테 주는게 맞는거 같은데, **Refresh 토큰**도 이렇게 줄지 고민이 됐다.  
여러 블로그들을 봤는데, 어떤 사람은 그냥 둘다 Response(JSON 형태)로 주고... 또 어떤 사람은 둘다 쿠키에 넣고... 어떤 사람은 Access 토큰은 Response에, Refresh 토큰은 쿠키에 넣더라ㅎㅎ  
대체 뭐가 더 좋은 방법일까?? 궁금해졌다. 그래서 바아로 구글링했다.  


결론은.. JWT로 보안성이 높은 로그인을 구현하려면,  
⭐**백엔드에서 프론트엔드로 Access Token은 JSON 형태로 넘겨주고, Refresh Token은 Cookie에 넣어주어야 한다**⭐  
아래 링크에 자세한 이유가 나와있다!  
https://medium.com/@uk960214/refresh-token-%EB%8F%84%EC%9E%85%EA%B8%B0-f12-dd79de9fb0f0  


그래서 나도 리프레시 토큰을 쿠키에 넣어주는 코드를 구현했다! (근데 이건 과제니까.. 리프레시 토큰도 JSON 응답에서 한눈에 보고 싶어서 JSON 응답에도 넣어줬다)  


이제, Postman으로 확인해보자!  
- 로그인 성공시 JSON 응답으로 access 토큰, refresh 토큰 둘다 잘 오는걸 확인 가능하다  
![image](https://user-images.githubusercontent.com/90256209/236616656-8ce25ea2-a412-4868-8b48-a951d15c52f2.png)  
- 쿠키에도 refresh 토큰이 잘 들어가 있다  
![image](https://user-images.githubusercontent.com/90256209/236616689-3158903d-eda2-4f05-a272-8c033323d83a.png)  


➡️ 발급 받은 토큰을 디코딩해보면, 유저의 id(pk)와 토큰 발급시간(iat), 토큰 만료시간(exp)을 볼 수 있다.  
내가 `2023-05-06 08:04:08' 에 로그인 했고,  
```  
SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=30),
```  
⬇️ 이렇게 토큰 유효기간을 30분으로 설정했기 때문에, 토큰 만료시간은 아래 사진처럼 `2023-05-06 08:34:08` 로 나오는게 맞다!!  
![image](https://user-images.githubusercontent.com/90256209/236616826-4b4bd354-0988-4f1a-be66-ce5841e31122.png)  


### 📌 Refresh 토큰을 통한 Access 토큰 재발급  
로그인할때 access 토큰, refresh 토큰을 발급해주는 걸로 끝내는게 아니라, **실제로 토큰이 만료되었을때 refresh 토큰으로 토큰을 재발급받는 기능**을 구현하고 싶어서 해봤다.  
대략적인 흐름은 `refresh 토큰이 유효한지 확인` → `refresh 토큰에 담긴 유저 id 로 유저 불러오기` → `그 유저로 다시 access 토큰 발급` 이렇다ㅎㅎ  
코드 설명은 주석으로 자세하게 해놓았다..!  
```  
class RefreshAccessToken(APIView):
    def post(self, request):
        # 쿠키에 저장된 refresh 토큰 확인
        refresh_token = request.COOKIES.get('refresh')

        if refresh_token is None:
            return Response({
                "message": "Refresh token does not exist"
            }, status=status.HTTP_403_FORBIDDEN)

        # refresh 토큰 디코딩 진행
        try:
            payload = jwt.decode(
                refresh_token, SECRET_KEY, algorithms=['HS256']
            )
        except:
            # refresh 토큰도 만료된 경우 에러 처리
            return Response({
                "message": "Expired refresh token, please login again"
            }, status=status.HTTP_403_FORBIDDEN)

        # 해당 refresh 토큰을 가진 유저 정보 불러 오기
        user = MyUser.objects.get(id=payload['user_id'])

        if user is None:
            return Response({
                "message": "User not found"
            }, status=status.HTTP_400_BAD_REQUEST)
        if not user.is_active:
            return Response({
                "message": "User is inactive"
            }, status=status.HTTP_400_BAD_REQUEST)

        # access 토큰 재발급 (유효한 refresh 토큰을 가진 경우에만)
        token = TokenObtainPairSerializer.get_token(user)
        access_token = str(token.access_token)

        return Response(
            {
                "message": "New access token",
                "access_token": access_token
            },
            status=status.HTTP_200_OK
        )
```  


➡️포스트맨으로 테스트 해봤더니 새로운 토큰이 잘 발급된다..! 이제 프론트에서는 이 새로운 토큰을 헤더에 넣어서 요청을 보내면 된다.  
![image](https://user-images.githubusercontent.com/90256209/236617483-aabc803a-4800-4e6b-9683-dbd3d9db60eb.png)  


### 📌 JWT Logout 구현하기  
로그아웃 로직은 이렇다.  
1️⃣ 프론트에서 LogoutApi를 호출한다.  
2️⃣ 호출과 동시에 프론트는 가지고 있던 Access token을 삭제한다.  
3️⃣ 백엔드에서는 cookie에 존재하는 Refresh token을 삭제한다.  
그래서 나는 쿠키의 Refresh 토큰을 삭제해주도록 구현했다. Postman으로 확인해보자ㅎㅎ  
![image](https://user-images.githubusercontent.com/90256209/236617700-df7ef90c-afe3-4c40-9541-757e38c4900d.png)  
➡️ 로그아웃이 잘되서 쿠키에 있던 refresh 토큰이 사라진다..!


### 📌 Permission 설정하기  
`permissions.py` 파일을 새로 만들어서 permission을 커스텀해주고, `community` 에 있는 게시판, 게시글 API에 적용해줬다.  
```  
class IsOwnerOrReadonly(permissions.BasePermission):
    def has_permission(self, request, view):
        # 로그인한 사용자인 경우 API 사용 가능
        return request.user and request.user.is_authenticated

    def has_object_permission(self, request, view, obj):
        # GET, OPTION, HEAD 요청일 때는 그냥 허용
        if request.method in permissions.SAFE_METHODS:
            return True
        # DELETE, PATCH 일 때는 현재 사용자와 객체가 참조 중인 사용자가 일치할 때만 허용
        return obj.myUser == request.user
```  
➡️ Postman으로 확인해보자. 게시판 조회 API에 JWT가 잘 적용되었는지 볼 것이다  
- 유효기간이 만료된 경우: 이렇게 친절하게 알려준다ㅎㅎ  
![image](https://user-images.githubusercontent.com/90256209/236618467-ceff90f0-a51a-4207-8db5-be76f02758a2.png)  
- 유효한 토큰으로 다시 요청을 보내면, 다시 잘 보인다!  
![image](https://user-images.githubusercontent.com/90256209/236618489-518bad5d-c67d-4a3f-8552-7d6a0c3c9f7b.png)  

---
## 🍀 느낀점  
***장고는 편리하다...*** 놀랐던게 장고에서는 클라이언트가 넘겨준 JWT로 유저를 불러오는걸 무려 **함수 하나**로 제공한다.. JWT를 추출해서, 파싱하고, 디코딩하고, 유저ID를 추출해서, 그 유저ID로 DB에서 유저 정보를 불러오는 로직을 내가 직접 클래스에 작성할 필요 없이 `authenticate()` 함수 하나로 그냥 끝나버리는 것... (약간 허무한거같기두 ㅎ)  

공식 문서에는 이렇게 나와있다.  


![image](https://user-images.githubusercontent.com/90256209/236618752-bd3c3149-b8b0-4272-a12a-02a942f8fe54.png)  


이번 기회로 로그인 및 사용자 인증에 대해 다시 자세히 복습해 볼 수 있어서 재밌었당!

---
## [6주차 - AWS EC2, RDS & Docker & Github Actions]  

## ✨Docker Compose✨  
Docker Compose란?  
내가 이해한 Docker Compose는 여러 이미지들에 대한 복잡한 run 명령어들을 docker-compose.yml에 작성해서 여러 컨테이너들을 한번에 실행시킬 수 있게 해주는 도구다.  

참고로 `docker-compose.yml` 에서 `expose:` 는 컨테이너의 포트번호를 알려주는 용도이다. 그럼 이게 `ports:` 랑 뭐가 다르지? 궁금해지는 게 당연하다ㅎㅎ  
둘의 차이점은 `ports:` 는 실제로 **외부**에서 접속할 때 **Host의 포트**와 **컨테이너의 포트**를 매칭시켜주지만, `expose:` 는 **내부**에서 사용되도록 **컨테이너의 포트**만 노출시키는 것이다.  

 ---
 ## ✨Github Actions의 Secrets✨  
`.gitignore` 에 `.env.prod` 를 추가하면, 배포할 때는 이 파일이 없어서 환경변수를 사용할 수 없다.  
그래서 깃허브는 Actions Secrets을 통해 환경 변수를 암호화해서 저장할 수 있는 기능을 제공한다.  
프로젝트 레포지토리의 [Settings] > [Secrets and variables] > [Actions] 에 들어가서 [New repository secret] 버튼을 눌러 배포할 때 필요한 환경변수들을 추가해주면, Actions가 실행될 때 환경변수 설정 파일이 자동으로 생성되는 것이다.  

---
## ✨로컬에서 Docker 컨테이너 실행하기✨  
여러 에러들을 겪었다ㅎㅎ..  


에러 1) 로컬에서 `docker-compose -f docker-compose.yml up --build` 로 웹 컨테이너랑 db 컨테이너 실행하려니까  
`django no module named 'rest_framework_simplejwt` 에러가 났다. Simple JWT 설치는 `requirements.txt` 에 있어서 당연히 자동으로 될줄 알았는데 왜 안되지? 하고 `requirements.txt` 를 다시 봤다.  
그랬더니,,ㅎ `djangorestframework-jwt==1.11.0` 이게 들어가 있더라ㅎㅎ 그래서 후딱 `djangorestframework-simplejwt==5.2.2` 로 수정해줬다!😎 (처음엔 `Dockerfile` 에 `RUN pip3 install djangorestframework-simplejwt` 를 추가해줬는데 이건 에러가 나더라..)  

에러 2) `ValueError: Related model 'account.myuser' cannot be resolved` ➡️ 구글링해보니 `AUTH_USER_MODEL = 'account.MyUser'` 로 설정한 모델이 가장 먼저 migrate 되어야 하는데 그냥 `python manage.py migrate` 를 하면, 다른 모델이 먼저 migrate 되서 발생하는 에러인것 같았다...  
그래서 `docker-compose.yml` 안에서 `python manage.py migrate account && [나머지 앱들] ...` 이렇게 바꿨더니 또 이미 만들어진 모델이라는 에러가 나서 이번에는 migrate 하는 코드를 다 지우고 `python manage.py runserver 0.0.0.0:8000` 만 남겼더니 드디어 성공했다..🫠🫠  


루트 URL에 아무것도 안만들어놔서 404 에러 페이지가 뜨지만 그래도 접속에 성공했다!  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/f830ae2b-ab6c-40a2-acec-8016552ffe57)  

---
## ✨실제 배포하기(with AWS, Github Actions, Docker)✨
### AWS EC2, RDS 구축  
EC2는 스토리지 크기를 최대인 30Gb로 설정해주고, 탄력적 IP를 할당해줬다.  
RDS는 MySQL 버전 5.7.41 로 설정해줬다.  

⬇️ EC2에서 RDS에 접속하려면, EC2의 보안그룹ID를 RDS의 보안그룹 인바운드 규칙에 추가해줘야 한다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/4e5c61da-f144-47f0-a114-521e39108b21)  


타임존, 인코딩 설정을 위해서 새로운 파라미터 그룹도 생성해줬다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/09db23da-7608-4bf7-af5c-699cf030eeec)  


`docker-compose.prod.yml` 에 포트포워딩이 80:80 으로 되어있길래 EC2 인바운드 규칙에 80번(HTTP) 포트도 추가해줬다. (추가로 HTTPS인 443번도 열어줬다.)  

### Github Actions 를 통한 배포  
에러1) 프로젝트에 `.env.prod` 파일을 만들어주고, Github에 가서 Actions Secrets 설정도 스터디 노션에 있는대로 쭉 진행해줬다. 이후 master 브랜치에 `git push origin shj718:master` 로 push 해줬는데도 workflow가 실행이 안됐다.  
이유는 `deploy.yml` 에 **dev 브랜치**에 push할 때 자동으로 배포되게 설정되어 있어서였다ㅎㅎ 다시 **master** 브랜치로 수정했다.  

에러2) `docker-compose.prod.yml` 에 `env_file` 설정이 `.env.prod` 가 아닌 `.env` 로 되어있어서 에러가 났다. 고쳐줬다.  


다 고쳐주니 잘 빌드 됐다!  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/d7e35fe7-fd50-402c-8448-1e916085f2ea)  

브라우저로 EC2에 접속해보면 서버가 떠있는걸 확인 가능🤗  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/813337f1-ff19-4c41-8e64-eda78c3d24c8)  

### API 요청 보내기  
JWT 없이 보냈으니 자격 인증 데이터가 없다는 응답이 오는게 당연하다.  
<img width="632" alt="image" src="https://github.com/shj718/django_rest_framework_17th/assets/90256209/2b30d18b-7536-4d5b-904a-d5b612e1e28d">  

근데 로그인 요청을 보냈더니 500 에러가 났다..  
<img width="634" alt="image" src="https://github.com/shj718/django_rest_framework_17th/assets/90256209/0d9a5d77-485b-4db0-9f37-c68143a6f42f">  


그동안 Spring으로 개발할때 500 Internal Server 에러가 나는건 경험상 8-90% 내 프로젝트 Service단에서 로직이 잘못된 경우였다.  근데 여기선 왜 나는지 모르겠어서 `docker logs --details 050f4c270e9f` 로 로그를 확인해봤다.  
<img width="946" alt="image" src="https://github.com/shj718/django_rest_framework_17th/assets/90256209/19f5674a-86b2-4959-ac48-4a79a72f9ac2">  
근데 뜬금없이 
```  
 /usr/local/lib/python3.8/site-packages/environ/environ.py:637: UserWarning: Error reading /home/app/web/venv/.env - if you're not configuring your environme
   warnings.warn(
```  
요런 에러 메세지가 나왔다.. 구글링해봤을때 `docker-compose.yml` 과 동일한 위치에 `.env` 파일을 두면 저절로 `docker-compose` 할 때 웹 애플리케이션에 `.env` 파일이 포함되지 않는다고 하던데.. 무슨 일인지 잘 모르겠다.  
에러 메세지를 복붙해서 구글링해도 똑같은 에러를 겪은 사람이 없어보인다. 그래서 `docker exec` 로 컨테이너 안에 들어가서 이것저것 많이 해봤는데 뭐가 문제인지 아직 모르겠다 하하... 혹시 이 에러의 이유를 아신다면 알려주세요🥲🥲  

---
## ✨회고✨  
확실히 도커는 이론으로 공부할때보다 실제 프로젝트에 적용하는게 어렵다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/ee5954b1-db9b-4c8a-9080-35adde77457e)  
[출처](https://velog.io/@kdh92417/Nginx%EC%99%80-Django-EC2%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-feat.-RDS-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EC%97%B0%EA%B2%B0)  
그래도 이번 기회를 통해 배포 아키텍쳐와 처음 써보는 Github Actions에 대해 공부해봐서 매우매우 유익했다..  

---
## [7주차 - AWS : https 인증]  
저 혼자 주차를 앞서나가네요.. (~~과제가 없던 중간고사 기간 주차를 빼먹었나봐요~~) 혼란을 드린다면 죄송합니다..ㅎㅎ  


## 💙개념부터 알아보자💙  
### HTTPS와 SSL  
`HTTPS`는 보안이 강화된 HTTP다. HTTP는 암호화되지 않은 방법으로 데이터를 전송하기 때문에 서버와 클라이언트가 주고 받는 메시지(Ex. 로그인할 때 비밀번호)를 감청하는 것이 매우 쉽다.  
`SSL 인증서`는 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서다. 이를 통해 통신 내용이 공격자에게 노출되는 것을 막을 수 있고, 클라이언트가 접속하려는 서버가 신뢰 할 수 있는 서버인지를 판단할 수 있다.  
따라서 정상적인 서비스라면 HTTPS와 SSL은 선택이 아닌 **필수**다.  


보통 서비스가 소규모라면, 1대의 서버에 Nginx 를 설치하고 Let's Encrypt 를 설치해서 SSL을 등록한다.  
다만 이럴 경우 트래픽이 늘어 로드밸런서 + 여러 서버 구성으로 확장하기가 쉽지 않다.  
그래서 우리는 ACM(AWS Certificate Manager) + Route 53 + ALB(Application Load Balancer) 를 사용할 것이다.  

### ALB(Application Load Balancer)  
ALB를 이용한 SSL 동작 과정은 이렇다.  

1) 서버로 request 가 들어오면 load balancer는 요청이 https(port 443) 요청인지 확인한다.  
2) 만약 http(port 80) 요청이면 load balancer 가 이 요청을 https 로 redirection 한다.  
https 요청이면 load balancer 가 SSL session 의 종단점 역할을 대신해 요청을 decryption 해 target group 의 80 번 포트로 요청을 forwarding 한다.  


이렇게 구성 하면 ec2 인스턴스에서 실행 중인 server가 ssl decryption 을 수행 하지 않아도 되니 조금더 가벼워 질수 있다. (내 서버의 짐을 줄여 준다.)  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/e4c94d49-c9ec-4034-a0d1-a6dda8a9718a)  

### Amazon Certificate Manager  
AWS에서 제공하는 SSL/TLS 인증서 관리 시스템  

### Route 53  
AWS에서 제공하는 DNS임. 도메인 네임 시스템(DNS)은 사람이 읽을 수 있는 도메인 이름(예: www.amazon.com)을 머신이 읽을 수 있는 IP 주소(예: 192.0.2.44)로 변환해주는 시스템  

### ELB(ALB)의 구성 요소  
ELB는 외부의 요청을 받아들이는 리스너(Listener)와 요청을 분산/전달할 리소스의 집합인 대상 그룹(Target Group)으로 구성된다. ELB는 다수의 리스너와 대상 그룹을 거느릴 수 있다.  


![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/07253625-8cce-45db-ab56-ccf1a90fa881)  

### 배포 점검 사항  
`DEBUG`  
운영서버에서는 절대로 디버깅 모드를 사용하지 않는다.  
```  
DEBUG = False
```  


`ALLOWED_HOSTS`  
디버깅 모드에서 ALLOWED_HOSTS 변수가 빈 리스트일 경우 ['localhost', '127.0.0.1', '[::1]'] 의미가 된다. 즉, 로컬 호스트에서만 접속이 가능하다.  
디버깅 모드를 끄면 일체 접속이 허용되지 않고 아래와 같이 명시적으로 지정한 호스트에만 접속할 수 있다.  
```  
ALLOWED_HOSTS = ['example.com', 'www.example.com', 'localhost', ]
```  

---
## 💙AWS를 이용한 HTTPS 적용💙  
### 1️⃣ SSL 인증  
1) 가비아에서 hyejun.store 도메인을 샀다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/2b25c87e-f962-46ca-8ad8-b25df06e1790)  


2) ACM에서 내 도메인에 대한 SSL 인증서를 받았다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/577d7e29-aff1-4a05-845f-73c58d567176)  


3) Route 53에 hyejun.store 에 대한 호스팅 영역 생성을 해주고, 다시 ACM으로 돌아가서 'Route 53 에서 레코드 생성' 눌러주면 된다.  


4) 가비아에 들어가서 네임서버를 AWS로 이관해줘야 한다. 빨간 영역의 네임서버 4개를 모두 추가해준다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/94748433-1315-470f-aff5-ffebeef5a86f)  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/da175281-32ee-491c-993b-9691b3a3d097)  


5) 인증서 발급 완료!  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/63d15f7a-4c44-404e-b34a-eb6840d2ceb4)  


### 2️⃣ 로드밸런서(ALB) 설정  
1) 대상 그룹을 만든다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/daef24d5-9a04-4600-ac47-8f27e7f3d108)  


2) 로드밸런서를 생성하고, 방금 만든 대상 그룹과 그에 대한 리스너 2개 (HTTP:80, HTTPS:443)를 추가해준다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/2c356a90-814c-408f-bee2-c64d5bf8a13a)  


3) 아까 발급한 SSL 인증서도 추가해준다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/ba35009c-3c12-48ba-bfd7-e7b0950149f4)  


4) HTTP → HTTPS 리다이렉션 설정 추가하기  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/040b946f-188c-4c6b-a299-754aa74dc853)


### 3️⃣ 로드밸런서를 Route 53의 도메인의 레코드에 등록  
루트, www 에 대한 A레코드를 각각 생성했다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/f6995231-e52c-4168-a1ff-87f01bff0ac3)  


### 4️⃣ ALB 타겟 그룹 Health Check 수정  
나는 루트 URL에 대해 아무것도 만들어놓지 않아서, 접속시 404 에러가 뜨기 때문에 Health Check Path가 루트(/)로 되어있고 Success Code가 200으로 되어있다면 Unhealthy로 뜰 수 밖에 없다.  

그래서 타겟 그룹으로 들어가서 Health Check Path를 바꿔주고 해당 URL로 접속하면 무조건 HTTP 200 코드를 주는 테스트용 API를 만들어놨다.  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/7dc7db46-8717-4172-b321-e57c28cfaad3)  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/cdd3d81f-09d6-4620-bc90-ec1d7830d89e)  






### 5️⃣ 결과 확인  
브라우저에 `http://hyejun.store`로 접속해도 `https://hyejun.store`로 잘 리다이렉션 된다🤗  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/d38a68df-a172-4167-923c-dfcd9d98fff9)  


Postman도 잘된다👍  
![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/47f1fcb4-40e4-4ab8-ae85-78472e54c2bf)  


---
## 🥲에러 해결 과정🥲  
에러 해결하는데 좀 많은 시간을 쓴거 같다ㅎㅎ  


HTTPS 적용이 안되는건 `ALLOWED_HOSTS=*`로 해결했고,  
저번에 까먹은 migrate을 하려는데 `env` 파일 관련 에러도 나고.. `Unknown MySQL server` 에러도 났다.  


나는 분명 `env.prod`에 RDS 설정을 다 제대로 해줬고..  
`entrypoint.prod.sh`에 이것도 추가해보고..  
```  
echo "Apply database migrations"
python manage.py makemigrations
python manage.py migrate
```  


`Dockerfile.prod`에 `jpeg-dev zlib-dev` 도 추가해보고..  

RDS 마스터 비밀번호를 아예 바꿔서 다시 설정해줘도..  


도저히 RDS에 migrate가 안되더라🥲 그래도 혹시나 하는 마음에 MySQL Workbench로 RDS에 접속해봤는데 테이블이 하나도 안만들어졌다.  


근데... 내가 `env.prod`에서 DB 이름, 마스터 유저 이름, 비밀번호, 포트, ALLOWED_HOSTS 다 계속 확인했는데 하나 확인 안한게 있더라...ㅎ  
알고보니까 맨 윗줄에 떡하니 있는 RDS 엔드포인트에 **다른 RDS**가 들어가 있었다...ㅎ  


바꿔주니까 바로 되더라ㅎㅎㅎㅎㅎ  

물론 아직 일부 테이블만 migrate되서 해결해야하는 상황이긴 한데 정말 너무너무 어이없었다ㅎ 앞으로는 에러가 나면 너무 당연하다고 생각되는 것부터 확인하자..  


---
##  💙회고💙  
이전에는 Nginx에서 직접 SSL인증서를 발급받는 것밖에 안해봐서 AWS가 SSL인증을 이렇게 지원하는지 전혀 몰랐다.. 앞으로 잘 써먹어야겠다  
서브도메인에 대해서도 SSL인증을 하느라 골머리를 앓은적이 있는데 역시 아마존 최고당  
그래서 이번 과제도 넘넘 유익했다  
다음부턴 RDS 엔트포인트 두번 세번 확인할거다ㅎㅎ  
벌써 기말고사라니 😵😵 그래두 다들 화이팅!  


![image](https://github.com/shj718/django_rest_framework_17th/assets/90256209/2d4f7f8a-da80-4647-a4b4-e2e605bf69eb)  
