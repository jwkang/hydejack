객체 직렬화 (Serializable)
===

# 개요
각 components 간에 데이터 전달에 관련해 `Intent`에서 `객체 직렬화`가 다뤄진다. 
`Activity` 뿐만 아니라 `Service`, `Broadcast Receiver` 역시 Intent를 사용하므로 객체 직렬화에 대한 이해가 필요하다. 
`AIDL`과 `Messenger 및 Handler`를 통한 `Binder 통신(IPC)`을 이해하기 위해서도 객제 직렬화에 대한 이해가 필요하다. 

Android API 문서를 보면 아래와 같이 `Parcelable` `interface`에 대한 내용도 볼 수 있다. 
    ![Image](https://i.imgur.com/t2IYvoP.png)
    ![Image](https://i.imgur.com/kfFRiiz.png)
뿐만 아니라, API 문서를 보면, Parcel, Bundle과 같이 직렬화와 관련된 객체를 사용하는 함수를 많이 볼 수 있다. 

이하에서는 객체 직렬화를 통해 `Binder`를 이용해 data를 전달하는 과정을 설명하고자 한다. 

# Serializable 객체
- 객체는 메모리에 존재할 것이고, 객체 자체는 함수와 변수들이 존재 한다. 
- 함수와 변수는 메모리의 다른 영역에 저장된다. (함수 : code 영역 , 변수 : heap/stack 영역)
- 직렬화는 전달할 데이터만 관심을 갖으므로, 함수는 무시하고 단지 변수만을 포함한다. 
    ![Image](https://i.imgur.com/9NVMmwC.png)

- 직렬화 개념은 **변수 데이터를 순차적으로 byte 코드로 나열하는 것**이다.
- byte 코드로 나열되면 이 데이터가 무엇이든 전송이 가능하다.
- `직렬화` 돼 전달된 객체는 수신측에서 다시 `역직렬화`돼 의미단위로 해석돼 사용할 수 있게된다. 
    ![Image](https://i.imgur.com/d8RTBAS.png)
- 서로 다른 Process들 끼리 `IPC`통신을 하기 위해서 Android에서는 `Binder`라는 것을 사용하는데, 이때 `Binder`는 `byte stream` 전송이 가능하므로 직렬화가 필요하다. 

## Serializable interface를 이용한 직렬화

직렬화 방법은 "Serializable" 을 상속 받으면 자동으로 직렬화 처리되어 객체가 생성된다.
"Serializable" 객체는 코드를 살펴보면 아래와 같이 아무것도 내용이 없다. 
```java
package java.io;

public interface Serializable {
}

```
이러한 `interface`를 `marker interface` 라고 한다. 
Serializable을 marker interface로 두는 이유는 해당 객체가 직렬화 가능한지를 marking하기 위함 이다. 
```java
    if (object_name instanceof Serializable) {
            // ...
    }
```
`Serializable`이라고 마킹해 두면 시스템 내부적으로 객체를 직렬화 시켜 저장하는 것이다.


직렬화 Class를 생성할때 꼭 지켜야할 조건이 있다.

1. 송신측과 수신측에 동일한 Class가 존재해야 한다.
2. 같이 패키지 명이 같아야 한다. (패키지 명이 다르면 송신측에서 보낸 객체가, 수신측에서 받을때 다른 객체라고 보고 역 직렬화를 할 수 없다)
   - Android Studio에서는 `File -> New -> Package`를 프로젝트에 Package를 추가할 수 있다. 
3. `Serializable`을 상속 받아야 한다.
4. `serialVersionUID`를 선언하고 같은 값을 가져야 한다. (이 값은 생략이 가능하지만 Warning이 발생한다)
    - `serialVersionUID`은 객체에 대한 버젼과 같다.
    - 이 버젼이 수신측과 송신측이 다르면 송신측에 보낸 객체를 수신측에서 받을때 객체는 동일한 객체로 보나, 객체 자체의 내용이 달라졌다고 보고 역직렬화를 하지 못한다.

## Serializable 객체를 통한 package간 통신 예제
| Server(caller) application 폴더 구성 | ![Image](https://i.imgur.com/odVPUrG.png) |
| ------------------------------------| --------------------------------------------- |
| Client(callee) application 폴더 구성 | ![Image](https://i.imgur.com/PFZRcVy.png) |

1. Serializable class
    ```java
    package com.test.serializableJava;

    public class SerializableJavaDataClass implements Serializable {
        public int m_value;
        public String m_name;
    }
    ```
    - 이 class는 `com.test.serializableJava`에 포함돼 있음을 주의
    - 이 class가 구현돼 있는 java 파일을 양 application이 모두 가지고 있어야 한다. 

2. Server측 전달하는 코드
   ```java
    private final String m_packageName = "jwkang2.superdroid.humaxdigital.com.clienttest";
    private final String m_classNameSerializable = "jwkang2.superdroid.humaxdigital.com.clienttest.SerializableTestActivity";

    private void SendDataViaSerializableObject() {
        Intent intent = new Intent();
        ComponentName cn = new ComponentName(m_packageName,
                m_classNameSerializable);
        intent.setComponent(cn);

        SerializableJavaDataClass data = new SerializableJavaDataClass();
        data.m_name = "Hello";
        data.m_value = 123;

        intent.putExtra(m_serializable_key, data);

        startActivity(intent);
    }
   ```
   - `Intent`를 통해 활성화할 Component는 `Activity`이며, 그 이름은 SerializableTestActivity 이다. 
   - Client측 SerializableTestActivity이 활성화 되며 `Intent`를 통해 data를 전달받는다.
   - 이때 전달되는 data는 `Serializable` instance가 되므로 system은 이 data를 직렬화 시킨다.
  
3. Client측 수신하는 코드 (`역 직렬화`)
   ```java
   public class SerializableTestActivity extends AppCompatActivity {

        private final String m_serializable_key = "jw_test";

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_serializable_test);

            Intent intent = getIntent();
            SerializableJavaDataClass data = (SerializableJavaDataClass) intent.getSerializableExtra(m_serializable_key);

            Log.i("tag", data.m_name);
            Log.i("tag", String.valueOf(data.m_value));

            TextView t1 = (TextView) findViewById(R.id.text_name);
            t1.setText(data.m_name);

            TextView t2 = (TextView) findViewById(R.id.text_value);
            t2.setText(new StringBuilder().append(data.m_value).toString());
        }   
   }
   ```

- Client 측 Activity가 호출되면 Server 측에서 전달한 intent 객체가 전달된다. 
- intent 객체내에는 직렬화 객체가 들어가 있다.
- 수신측에서는 이 객체를 읽어들어 객체의 내용을 출력할 수 있다.

자료 참조 : <http://cafe.daum.net/superdroid/aAfL/77>