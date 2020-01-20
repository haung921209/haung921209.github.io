---
layout: post
title:  "markdownTest"
date:   2020-01-20T17:08:52-05:00
author: Hyungmin Oh
categories: 생활

---

# Chap 5. 대규모 개발에서도 통용되는 작성법 익히기 - 객체지향 구문


1. JavaScript의 객체지향 특징
    1. 클래스는 없고 프로토타입만 있다
        - '인스턴스화 및 인스턴스'라는 개념은 존재하나 클래스가 없고, '프로토타입(모형)' 개념만 존재
    2. 가장 간단한 클래스 정의하기

            var Member = function() {}; //자바스크립트의 Member 클래스
            var mem = new Member(); // new 연산자로 인스턴스화

        - JavaScript에서는 함수(Function 객체)에 클래스의 역할을 부여한다.
        - 에로우 함수에서는 생성자를 정의할 수 없다(ES2015)

                let Member = () => { ... 생성자의 내용 ...};
                let m = new Member(); // Error : Member is not a constructor

            (ES2015) 환경에서는 순수하게 class 명령을 이용해야 한다.

    3. 생성자로 초기화하기

            var Member = function(firstName, lastName) {
            	this.firstName = firstName;
            	this.lastName = lastName;
            	this.getName = function() {
            		return this.lastName + ' '+ this.firstName;
            	}
            };
            var mem = new Member('철수', '강');
            console.log(mem.getName()); //결과 : 강 철수

        - this 키워드는 생성자에 의해 생성되는 인스턴스를 나타냄.
        - 함수 객체로서의 값이 프로퍼티의 메소드로 간주됨.
    4. 동적으로 메소드 추가하기
        - 메소드를 나중에 정의하는 방식

                var Member = function(firstName, lastName) {
                	this.firstName = firstName;
                	this.lastName = lastName;
                }
                
                var mem = new Member('철수', '강');
                mem.getName = function() {
                	return this.lastName + ' ' + this.firstName;
                }//생성된 인스턴스에 대해 메소드가 추가되고 있음.
                console.log(mem.getName());// 결과 : 강 철수

        - 인스턴스에 대해서 직접 멤버(프로퍼티 or 메소드)를 추가하는 방식

                var Member = function(firstName, lastName) {
                	this.firstName = firstName;
                	this.lastName = lastName;
                }
                var mem = new Member('철수', '강');
                
                mem.getName = function() {
                	return this.lastName + ' ' + this.firstName;
                }
                console.log(mem.getName()); //결과 : 강 철수
                
                var mem2 = new Member('영희', '이');
                console.log(mem2.getName()); // error : mem2.getName is not a function

    5. 문맥에 따라 내용이 변하는 변수 - this 키워드
        1. this 키워드가 참조하는 곳
            - 톱 레벨(함수의 바깥) - 글로벌 객체
            - 함수 - 글로벌 객체(strict 모드에서는 undefined)
            - call/apply 메소드 - 인수로 지정된 객체

                    func.call(that [,arg1 [,arg2 [,...]]])
                    func.apply(that [,args])
                    /*
                    func : 함수 객체
                    that : 함수 안에서 this 키워드가 가리키는 것
                    			 참고로 인수 that에 null을 건넬경우, 암묵적으로 글로벌 객체가 건네진 것으로 간주.
                    arg1, arg2, ... : 함수에 건넬 인수
                    args : 함수에 건넬 인수(배열)
                    */

                    //call method 예시. apply 메소드의 경우도 동일함.
                    var data = 'Global data';
                    var obj1 = { data: 'obj1 data' };
                    var obj2 = { data: 'obj2 data' };
                    
                    function hoge() {
                    	console.log(this.data);
                    }
                    
                    hoge.call(null); //결과 : Global data
                    hoge.call(obj1); //결과 : obj1 data
                    hoge.call(obj2); //결과 : obj2 data

                    //배열과 비슷하지만 배열이 아닌 객체를 배열로 변환하기
                    function hoge() {
                    	//arguments 객체를 this로 해서 Array.slice 객체를 호출하시오.
                    	var args = Array.prototype.slice.call(arguments);
                    	console.log(args.join('/'));
                    }
                    hoge('Angular', 'React', 'Backbone'); // 결과 : Angular/React/Backbone

            - 이벤트 리스너 - 이벤트의 발생처
            - 생성자 - 생성한 인스턴스
            - 메소드 - 호출원의 객체(=리시버 객체)
    6. 생성자의 강제적인 호출

            var Member = function(firstName, lastName) {
            	this.firstName = firstName;
            	this.lastName = lastName;
            };
            
            var m = Member('인식', '정');
            console.log(m); //결과 : undefined
            console.log(firstName); //결과 : 인식
            console.log(m.firstName); // 결과 : 에러(Cannot read property 'firstName' of undefined)

        이 경우 Member 객체는 생성되지 않고, 대신 글로벌 변수로 firstName/lastName이 생성되어 버린다(this가 글로벌 객체를 나타내고 있기 때문)

        따라서

            var Member = function(firstName, lastName) {
            	if(!(this instanceof Member)) {//여기서 this는 Member 객체가 아니라 글로벌 객체. 
            		return new Member(firstName, lastName);//따라서 아닌 경우 new 연산자로 호출.
            	}
            	this.firstName = firstName;
            	...중략...
            };

2. 생성자의 문제점과 프로토타입

     0.  생성자에 의한 메소드 추가 → 메소드의 수에 비례하여 '쓸데없이' 메모리를 소비한다 

    1. 메소드는 프로토타입으로 선언한다 - prototype 프로퍼티
        - 객체를 인스턴스화 했을 경우, 인스턴스는 베이스가 되는 객체에 속하는 prototype 객체에 대해서 암묵적인 참조를 갖게 된다.

                var Member = function(firstName, lastName) {
                	this.firstName = firstName;
                	this.lastName = lastName;
                }
                
                Member.prototype.getName = function() {
                	return this.lastName + ' ' + this.firstName;
                
                var mem = new Member('인식' + '정');
                console.log(mem.getName()); // 결과 : 정 인식
                // prototype 객체에 추가된 getName 메소드가 Member 클래스의 인스턴스(변수 mem)에서도 올바르게 참조됨.

    2. 프로토타입 객체를 사용한 메소드 정의의 두 가지 이점
        1. 메모리의 사용량을 절감할 수 있다.
            - JavaScript에서는 객체의 멤버가 호출되었을 때 다음의 흐름으로 멤버를 취득
                - 인스턴스 측에 요구된 멤버가 존재하지 않는지 확인
                - 존재하지 않는 경우에는 암묵적인 참조를 통해 프로토타입 객체를 검색

                → 생성자 경유로 메소드 정의시 발생하는 메모리 낭비 문제 회피.

        2. 멤버의 추가나 변경을 인스턴스가 실시간으로 인식할 수 있다.
            - 인스턴스에 멤버를 복사하지 않는다 라고 하는 것은 프로토타입 객체로의 변경(추가나 삭제)을 인스턴스 측에서 동적으로 인식할 수 있다 라는 뜻도 된다.

                var Member = function(firstName, lastName){
                	this.firstName = firstName;
                	this.lastName = lastName;
                }
                
                var mem = new Member('인식', '정');
                
                //인스턴스화 후에 메소드를 추가
                Member.prototype.getName = function() {
                	return this.lastName + ' ' + this.firstName;
                };
                
                console.log(mem.getName());//결과 : 정 인식
                //인스턴스 생성 후에 getName 메소드 추가 -> 아무 문제 없이 메소드를 인식함.

    3. 프로토타입 객체의 불가사의(1) - 프로퍼티의 설정

            var Member = function() { };
            
            Member.prototype.sex = '남자';
            var mem1 = new Member();
            var mem2 = new Member();
            
            console.log(mem1.sex + '|' + mem2.sex);// 결과 : 남자|남자
            mem2.sex = '여자';
            console.log(mem1.sex + '|' + mem2.sex);// 결과 : 남자|여자

        → 프로토타입 객체가 사용되는 것은 '값을 참조할 때 뿐'이다. 값의 설정은 항상 인스턴스에 대해 행해진다.

    4. 프로토타입 객체의 불가사의(2) - 프로퍼티의 삭제

            var Member = function() { };
            
            Member.prototype.sex = '남자';
            var mem1 = new Member();
            var mem2 = new Member();
            
            console.log(mem1.sex + '|' + mem2.sex);// 결과 : 남자|남자
            mem2.sex = '여자';
            console.log(mem1.sex + '|' + mem2.sex);// 결과 : 남자|여자
            
            delete mem1.sex
            delete mem2.sex
            console.log(mem1.sex + '|' + mem2.sex); // 결과 : 남자 | 남자

        → 인스턴스 측에서의 멤버 추가나 삭제 조작이 프로토타입 객체에까지 영향을 미칠 일은 없다.

        - 인스턴스 단위로 프로토타입 멤버 삭제하기

                delete Member.prototype.sex

            → 참조하고 있는 모든 인스턴스에 영향을 미친다.

            - 프로토타입에서 정의된 멤버를 인스턴스 단위로 삭제하고 싶은 경우 → undefined를 이용

                    var Member = function() {};
                    Member.prototype.sex = '남자';
                    
                    var mem1 = new Member();
                    var mem2 = new Member();
                    console.log(mem1.sex + '|' + mem2.sex); // 결과 : 남자|남자
                    mem2.sex = undefined;
                    console.log(mem1.sex + '|' + mem2.sex); // 결과 : 남자|undefined
                    // 덮어씀으로써 '유사적으로' 멤버를 무효화

    5. 객체 리터럴로 프로토타입 정의하기
        - 지금까지의 방법

                var Member = function(firstName, lastName){
                	this.firstName = firstName;
                	this.lastName = lastName;
                };
                
                Member.prototype.getName = function(){
                	return this.lastName + ' ' + this.firstName;
                };
                
                Member.prototype.toString = function(){
                	return this.lastName + this.firstName;
                };

            → 귀찮고, 가독성이 좋지 않음

        - 객체 리터럴을 이용하는 방법

                var Member = function(firstName, lastName){
                	this.firstName = firstName;
                	this.lastName = lastName;
                };
                
                Member.prototype = {
                	getName : function(){
                		return this.lastName + ' ' + this.firstName;
                	},
                toString : function(){
                	return this.lastName + this.firstName;
                	}
                };

            - 얻을 수 있는 효과
                - 'Member.prototype.~'와 같은 기술이 최소한으로 억제된다.
                - 그 결과, 객체명의 변경이 있을 경우에도 영향 받는 범위는 한정 가능하다
                - 동일 객체의 멤버 정의가 하나의 블록에 담겨져 있기 때문에 코드의 가독성이 향상된다.
    6. 정적 프로퍼티 / 정적 메소드 정의하기
        - 정적 프로퍼티 / 정적 메소드란 인스턴스를 생성하지 않아도 객체로부터 직접 호출할 수 있는 프로퍼티 / 메소드이다.

                //정적 프로퍼티/정적 메소드는 다음과 같이 생성자에 직접 추가한다
                객체명.프로퍼티명 = 값
                객체명.메소드명 = function() { /*메소드의 정의*/ }

        - 정적 프로퍼티 / 정적 메소드를 정의할 떄 두 가지 주의점
            - 정적 프로퍼티는 기본적으로 읽기 전용의 용도로 사용
            - 정적 메소드 안에서는 this 키워드를 사용할 수 없다.
        - 왜 정적 멤버를 정의하는 것인가?

            → 글로벌 변수 / 함수는 이름이 충돌하는 원인이 된다. 그러기 때문에 가능한 적게 사용해야 한다. 이를 위해 정적 멤버를 정의함으로서 사용을 적게 하는 역할을 하게 된다.

3. 객체의 계승 - 프로토타입 체인
    1. 프로토타입 체인의 기초

            var Animal = function() {};
            
            Animal.prototype = {
            	walk : function() {
            		console.log('종종...');
            	}
            };
            
            var Dog = function() {
            	Animal.call(this);
            };
            
            Dog.prototype = new Animal();//Dog 객체의 프로토타입이 Animal 객체의 인스턴스로 세트
            Dog.prototype.bark = function(){
            	console.log('멍멍!! ');
            }
            
            var d = newDog();
            d.walk();
            d.bark();

        호출의 흐름

        1. Dog 객체의 인스턴스 d로부터 멤버의 유무를 검색한다
        2. 해당하는 멤버가 존재하지 않는 경우에는 Dog 객체의 프로토타입 - 즉 Animal 객체의 인스턴스를 검색한다
        3. 거기서도 목적의 멤버가 발견되지 않은 경우에는 한층 위의 Animal 객체의 프로토타입을 검색한다.(예제)
        4. (만약 여기서도 발견되지 않은 경우) 한층 더 위의 프로토타입(구체적으로는 최상위의 Object.prototype까지)으로 거슬러 올라가게 된다.
    2. 계승 관계는 동적으로 변경 가능

            var Animal = function() {};
            
            Animal.prototype = {
            	walk : function() {
            		console.log('종종...');
            	}
            };
            
            var SuperAnimal = function() {};
            SuperAnimal.prototype = {
            	walk : function() {
            		console.log('다다다닷!');
            	}
            };
            
            var Dog = function() {};
            Dog.prototype = new Animal();
            var d1 = newDog();
            d1.walk(); // 결과 : 종종...
            
            Dog.prototype = new SuperAnimal(); //SuperAnimal 객체를 계승
            var d2 = new Dog();
            d2.walk(); //결과 : 다다다닷!
            d1.walk(); //결과 : ????? -> 결과는 종종...

        → JavaScript의 프로토타입 체인은 인스턴스가 생성된 시점에서 고정되어 그 후의 변경에는 관여치 않고 보존된다.

    3. 객체의 타입 판정하기
        - 스크립트상에서 다루고 있는 타입을 판정하는 방법
            1. 바탕이 되는 생성자 취득하기 - constructor 프로퍼티

                    //Animal 클래스와 이것을 계승한 Hamster 클래스 준비
                    var Animal = function() {};
                    var Hamster = function() {};
                    Hamster.prototype = new Animal();
                    
                    var a = new Animal();
                    var h = new Hamster();
                    console.log(a.constructor === Animal); // 결과 : true
                    console.log(h.constructor === Animal); // 결과 : true
                    console.log(h.constructor === Hamster); // 결과 : false

                → 프로토타입 계승을 하는 경우에는 constructor 프로퍼티가 나타내는 것이 상속원의 클래스(예제에서는 Animal 클래스)가 된다.

            2. 바탕이 되는 생성자 판정하기 - instanceof 연산자
                - 객체가 특정 생성자에 의해 생성된 인스턴스인지의 여부를 판정

                    console.log(h instanceof Animal); //결과 : true
                    console.log(h instanceof Hamster); //결과 : true

                → instanceof 연산자는 본래의 생성자(Hamster)는 물론이고 프로토타입 체인을 거슬러 올라가는 판정도 가능하다.(이 예에서는 Animal)

            3. 참조하고 있는 프로토타입 확인하기 - isPrototypeOf 메소드
                - 객체가 참조하는 프로토타입을 확인하는 데 이용한다.

                    console.log(Hamster.prototype.isPrototypeOf(h)); //결과 : true
                    console.log(Animal.prototype.isPrototypeOf(h)); //결과 : true

            4. 멤버의 유무 판정하기 - in 연산자

                    var obj = { hoge: function(){}, foo: function(){} };
                    
                    console.log('hoge' in obj); // 결과 : true
                    console.log('piyo' in obj); // 결과 : false

4. 본격적인 개발에 대비하기
    1. private 멤버 정의하기
        - JavaScript에서는 private 멤버를 정의하기 위한 구문은 없으나, 클로저를 이용해 유사하게 구현 가능.

                function Triangle(){
                	//(1)scope start
                	//private 프로퍼티의 정의(밑변/높이를 보존)
                	var _base;
                	var _height;
                	// private 메소드의 정의(인수가 올바른 숫자인가를 체크)
                	var _checkArgs = function(val){
                		return (typeof val === 'number' && val > 0);
                	}
                	//(1)scope end
                
                	//(2)scope start
                	//private 멤버에 엑세스하기 위한 메소드의 정의
                	this.setBase = function(base){
                		if(_checkArgs(base)){ _base = base;}
                	}
                	this.getBase = function() {return _base;}
                	
                	this.setHeight = function(height){
                		if(_checkArgs(height)){ _height = height; }
                	}
                	this.getHeight = function() { return _height; }
                	//(2)scope end
                }
                
                //private 멤버에 액세스하지 않는 보통의 메소드를 정의
                Triangle.prototype.getArea = function() {
                	return this.getBase() * this.getHeight() / 2;
                }
                
                var t = new Triangle();
                t._base = 10;
                t._height = 2;
                console.log('삼각형의 면적 : ' + t.getArea()); // 결과 : NaN
                
                t.setBase(10);
                t.setHeight(2);
                console.log('삼각형의 밑변 : ' + t.getBase()); //결과 : 삼각형의 밑변 : 10
                console.log('삼각형의 높이 : ' + t.getHeight()); //결과 : 삼각형의 높이 : 2
                console.log('삼각형의 면적 : ' + t.getArea()); // 결과 : 삼각형의 면적 : 10

            → _base/ _height 프로퍼티, _checkArgs 메소드가 private 멤버이다.

            1. private 멤버는 생성자 함수에서 정의한다.

                    //private 프로퍼티/메소드
                    var 프로퍼티명
                    var 메소드명 = function(인수, ...) { ... 임의의 처리 ...} 

                → private 멤버를 정의하는 경우 this 키워드가 아니라 var 키워드로 선언한다.

            2. 'privileged 메소드'를 정의해 private 멤버에 엑세스하기
                - private 멤버에 엑세스할 수 잇는 메소드를 privileged 메소드라고 한다. '생성자 함수 안에서 정의한다'는 점만 빼면 일반적인 메소드와 같은 방법으로 정의 가능함.
        - 액세서 메소드 경유로 프로퍼티 공개하기
            - 프로퍼티 그 자체는 클래스 외부로부터 엑세스 할 수 없게 해두고, 대신에 프로퍼티에 엑세스하기 위한 메소드( = 액세서 메소드(참조용 : 게터 메소드, 설정용 : 세터 메소드))
            - 사용하는 경우
                - 값을 읽기(쓰기) 전용으로 하고 싶다.
                - 값을 참조할 때 데이터를 가공하고 싶다.
                - 값을 설정할 때 타당성을 검증하고 싶다.
            - 일반적으로 'get + 프로퍼티명', 'set + 프로퍼티명'의 형식으로 명명함. 프로퍼티명의 머리글자는 대문자로 한다.
    2. Object.defineProperty 메소드에 의한 액세서 메소드 구현

            //defineProperty 메소드
            Object.defineProperty(obj, prop, desc)
            	obj : 프로퍼티를 정의하는 객체
            	prop : 프로퍼티명
            	desc : 프로퍼티의 구성 정보

        → defineProperty 메소드는 인수 desc에 대해서 get/set 파라미터를 지정함으로써 게터/세터를 정의할 수 있음.

            function Triangle() {
            	// private 변수를 선언
            	var _base;
            	var _height;
            
            	// base 프로퍼티를 정의
            	Object.defineProperty(
            		this,
            		'base',
            		{
            			get: function() {
            				return _base;
            			}
            			set: function(base) {
            				if(typeof base === 'number' && base > 0) {
            					_base = base;
            				}
            			}
            		}
            	);
            	
            
            	//height 프로퍼티를 정의
            	Object.defineProperty(
            		this,
            		'height',
            		{
            			get: function(){
            				return _height;
            			}
            			set: function(){
            				if(typeof height === 'number' && height > 0 ) {
            					_height = height;
            				}
            			}
            		}
            	);
            };
            
            Triangle.prototype.getArea = function() {
            	return this.base * this.height / 2 ;
            };
            
            var t = new Triangle();
            t.base = 10;
            t.height = 2;
            console.log('삼각형의 밑변 : ' + t.base); // 결과 : 삼각형의 밑변 : 10
            console.log('삼각형의 높이 : ' + t.height); // 결과 : 삼각형의 높이 : 2
            console.log('삼각형의 면적 ; ' + t.getArea); // 결과 : 삼각형의 면적 : 10

            //get/set 파라미터
            
            get: function() {
            	return private변수
            },
            set: function(value){
            	private변수 : value
            }

        - 여러 프로퍼티를 함께 정의하기

                //defineProperties 메소드
                Object.defineProperties(obj, props)
                	
                	obj : 프로퍼티를 정의하는 객체
                	props : 프로퍼티의 구성 정보("프로퍼티명: 구성 정보"의 해시 형식)

                // 위의 예시를 defineProperties 메소드 이용하여 고쳐 쓴 예제
                Object.defineProperties(this, {
                	base:{
                		get: function() {
                			return _base;
                		},
                		set: function(base){
                			if(typeof base === 'number' && base > 0){
                				_base = base;
                			}
                		}
                	},
                	height: {
                		get:function() {
                			return _height;
                		},
                		set: function(height){
                			if(typeof height === 'number' && height > 0){
                				_height = height;
                			}
                		}
                	}
                });

    3. 네임스페이스 / 패키지 작성하기
        - JavaScript에는 namespace가 없음 → 빈 객체를 이용 유사 네임스페이스를 만들어 활용함.

                var Wings = Wings || {};//Wings가 미정의인 경우에만 새롭게 네임스페이스를 작성한다.
                
                Wings.Member = function(firstName, lastName){
                	this.firstName = firstName;
                	this.lastName = lastName;
                };//Wings 네임스페이스 하에 속하는 Member 클래스 정의됨.
                
                Wings.Member.prototype = {
                		getName : function() {
                		return this.lastName + ' ' + this.firstName;
                	}
                };
                
                var mem = new Wings.Member('인식', '정');
                console.log(mem.getName());

        - 계층을 지닌 네임스페이스 정의하기
            - 위의 예시를 반복함으로서 정의해도 되지만, 네임스페이스를 위한 함수를 작성하면 편의성 증가

                    function namespace(ns){
                    	// 네임스페이스를 '.' 구분으로 분할
                    	var names = ns.split('.');
                    	var parent = window;
                    
                    	//namespace를 상위부터 순서대로 등록
                    	for(var i = 0, len = names.length; i<len; i++){
                    		parent[names[i]] = parent[names[i]] || {};
                    		parent = parent[names[i]];
                    	}
                    	return parent;
                    }
                    
                    //Wings.Gihyo.Js.MyApp 네임스페이스를 등록
                    var my = namespace('Wings.Gihyo.Js.MyApp');
                    my.Person = function() {};
                    var p = new my.Person();
                    console.log(p instanceof Wings.Gihyo.Js.MyApp.Person); // 결과 : true

            → Wings.Gihyo.Js.MyApp.Person을 my.Person(별명)으로 표현할 수 있게 됨.

5. ES2015의 객체지향 구문
    1. 클래스 정의하기 - class 명령(ES2015)
        - firstName/lastName 프로퍼티, getName 메소드를 가지는 Member 클래스를 class 명령을 이용해 정의하기

                class Member {
                	//생성자
                	constructor(firstName, lastName){
                		this.firstName = firstName;
                		this.lastName = lastName;
                	}
                
                	//메소드
                	getName() {
                		return this.lastName + this.firstName;
                	}
                }
                let m = new Member('시온', '정');
                console.log(m.getName()); //결과 : 정시온
                
                
                /*
                class 명령
                class 클래스명 {
                	...생성자의 정의...(constructor로 고정)
                	...프로퍼티의 정의...
                	...메소드의 정의...
                }
                
                생성자/메소드의 정의
                메소드명(인수, ...) {
                	... 메소드의 본체 ...
                }
                */
                		

        - 클래스는 function 생성자와 다르다.
            1. 함수로서 호출할 수 없다.

                    let m = Member('시온', '정');//new 연산자가 없다
                    //결과 : Class constructor Member cannot be invoked without 'new'

            2. 정의하기 전의 클래스를 호출할 수 없다.

                    let m = new Member('시온', '정');
                    // 결과 : Member is not defined
                    class Member{ ...생략...}

        - 프로퍼티 정의하기
        - 정적 메소드 정의하기
        - 기존 클래스 계승하기
        - 기본 클래스의 메소드/ 생성자 호출하기 - super 키워드
    2. 객체 리터럴의 개선(ES2015)
    3. 애플리케이션을 기능 단위로 모으기 - 모듈(ES2015)
    4. 열거 가능한 객체 정의하기 - 반복자(ES2015)
    5. 열거 가능한 객체를 더욱 간단하게 구현하기 - 발생자(ES2015)
    6. 객체의 기본적인 동작을 사용자 정의하기 - Proxy 객체(ES2015)