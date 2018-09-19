Intent객체 직렬화 (Parcelable 과 Bundle)
===

`Parcelable` 객체는 프로세스간 데이터(객체 포함)를 전달하는 목적으로 존재한다.
IPC 측면에서는 `Serializable` 개념과는 크게 다르지 않으나, Serializable 객체는 Java standard로 사용되는 반면, Android에서는 Parcelable 객체를 주로 IPC를 위해 사용한다. 

> Serializable is a standard Java interface. You simply mark a class Serializable by implementing the interface, and Java will automatically serialize it in certain situations. Parcelable is an Android specific interface where you implement the serialization yourself. ... However, you can use Serializable objects in Intents.

# Pacel

`Parcelable` 객체를 이해하기 위해서는 Parcel이라는 객체를 먼저 이해해야 한다.
**Parcel 객체는 Process간 전달한 데이터를 묶는 데이터 컨테이너**라고 볼 수 있다.
프로세스 통신을 위해 데이터는 직렬화 되어야 하며, Parcel 객체는 직렬화한 데이터 그 자체이다.

![Image](https://i.imgur.com/W3GGTWl.png)
위의 그림과 같이 Process 간 통신을 IPC라고 하고 IPC 대상이 되는 데이터가 `Parcel`이다.
다시 말해 Parcel은 송신측에서 수신측으로 전달되는 데이터를 저장하는 용도이다.

- IPC 통신은 최대한 고속으로 이뤄져야 한다. 그러므로 전달되는 데이터는 IPC 통신이 신속히 이뤄질 수 있도록 최적화 되어야 하고 고성능 IPC 전송을 가능케 하는 데이터가 바로 Parcel이다.
- <http://www.developerphil.com/parcelable-vs-serializable/>
  ![Image](https://i.imgur.com/MfGVzog.png)

## Pacel의 제약 사항
- `Parcel` 데이터를 추가한 순서대로 읽어야 한다. 추가한 순서대로 읽지 못하면 data가 깨지게 된다. 
    ```java
        Parcel pl = Parcel.obtain();
        pl.writeString("Hello");
        pl.writeInt(123);
        pl.writeFloat(3.14f);

        pl.setDataPosition(0);

        Log.i(TAG, "SendDataViaParcelable: pl.readString()" + pl.readString());
        Log.i(TAG, "SendDataViaParcelable: pl.readInt()" + String.format("%d", pl.readInt()));
        Log.i(TAG, "SendDataViaParcelable: pl.readFloat()" + String.format("%f", pl.readFloat()));
    ```
- `setDataPosition()` 함수는 Pacel이 참조하는 Position을 변경해 주는 함수 이다. 
- 결론적으로 `Parcel`을 통해 데이터를 저장하면, 그 컨테이너를 통해 Binder를 통해 더 고성능의 data 전송은 가능하나, 순서대로 데이터를 꺼내야 한다는 단점이 있다.
- `Parcelable`객체는 데이터에 대한 조립과 분해를 쉽게할 수 있도록 도와준다. 

# Parcelable
![Image](https://i.imgur.com/R06ETBI.png)

![Image](https://i.imgur.com/VnzWRx1.png)

`Parcel` 객체는 직렬화 class로써 내부 데이터들이, 모두 `Serializable` 객체를 이용해서 직렬화 되어 있고, `Parcel` 객체의 분해와 조립을 쉽게 해주는 것이 `Parcelable` 객체이다.

`Parcelable` 객체에서 아래 함수가 가장 중요하다.
1. `writeToParcel` : Parcel 객체를 이용해 전달할 데이터를 분해하는 함수
   `public void writeToParcel(Parcel dest, int flags)`

2. `createFromParcel` : 전달 받은 Parcel 객체 데이터를 조립하는 함수  
   `public SampleParcelable createFromParcel(Parcel src)` 

- Parcelable 객체의 writeToParcel, createFromParcel에 Parcel에 데이터를 추가한 순서대로 데이터를 쓰고 읽으므로서, 쉽게 순서에 맞는 Pacel 데이터 전송이 가능하다. 
- 실제 객체와 조립을 하는 함수 `writeToParcel`, `createFromParcel` 은 직렬화/역직렬화 과정 중 Android Framework에 의해 호출된다. 
- Parcelable class 예
    ```java
    public class SampleParcelable implements Parcelable {
        static final String TAG = "SampleParcelable";
        
        public int 		mIntData1;
        public String 	mStringData2;
        public float   	mFloatData3;
        
        public SampleParcelable(int intData, String stringData, float floatData)    {
            Log.i(TAG, "SampleParcelable()");
            
            mIntData1		= intData;
            mStringData2	= stringData;
            mFloatData3		= floatData;
        }

        protected SampleParcelable(Parcel in) {
            mIntData1 = in.readInt();
            mStringData2 = in.readString();
            mFloatData3 = in.readFloat();
        }

        public static final Creator<SampleParcelable> CREATOR = new Creator<SampleParcelable>() {
            @Override
            public SampleParcelable createFromParcel(Parcel in) {
                return new SampleParcelable(in);
            }

            @Override
            public SampleParcelable[] newArray(int size) {
                return new SampleParcelable[size];
            }
        };

        @Override
        public void writeToParcel(Parcel dest, int flags) {
            Log.i(TAG, "writeToParcel()");

            dest.writeInt(mIntData1);
            dest.writeString(mStringData2);
            dest.writeFloat(mFloatData3);
        }

        @Override
        public int describeContents() 
        {
            Log.i(TAG, "describeContents()");
            return 0;
        }
    }

    // Pacelable 객체를 Intent를 통해 보내는 예
    private void SendDataViaParcelable()
    {
        Intent intent = new Intent();
         ComponentName cn = new ComponentName(m_packageName,
                 m_classNameParcelable);
        intent.setComponent(cn);

        SampleParcelable sp = new SampleParcelable(123, "Hello ParcelWorld", 3.14f);
        intent.putExtra(m_parcle_key, sp);        
        
        startActivity(intent);
    }
    ```

    Pacelable 객체는 다음과 같이 빠르게 생성할 수도 있다.
    ![Image](https://i.imgur.com/Xd7rkzJ.gif)

# 멀티 프로세스와 Bundle 
안드로이드는 기본적으로 멀티 프로세스 환경이기 이다. 
프로세스 끼리는 메모리를 공유하지 않기 때문에 안드로이드에서는 직렬화시켜 서로 데이터를 주고 받는다. 
안드로이드용 직렬화 객체는 `Parcel` 이다. `Parcel`은 분해와 조립의 순서가 매우 중요하기 때문에 `Parcelable` 클래스를 상속받아 순서에 맞게 조립(`writeToParcel`)과 분해(`createFromParcel`)를 가능하게 한다.

`Parcelable` 객체 역시 조립 분해 method를 구현해야 하기 때문에 번거롭다는 점이 있다. 
`primitive 타입`(`int`, `String` 등)의 데이터를 전송할때 조립 분해 순서를 생각치 않고 사용하는 객체가 있을면 편리할 것 이다.
이 것이 `bundle`이다. `bundle`은 `parcelable`을 상속받아 편리하게 사용하기 위한 클래스이다. 
**즉 bundle도 Parcelable 이다.**

## Bundle 객체

`Parcel` 객체는 `직렬화 Class`로써 내부 데이터들이 모두 `Serializable 객체`를 이용해서 `직렬화` 되어 있고, `Parcel` 객체의 분해와 조립을 처리하는 `Parcelable` 객체가 존재한다.

`Bundle`은 `Parcelable` 객체를 상속받아 구현한 객체 이다. 

![Image](https://i.imgur.com/6rsmLxa.png)

- Bundle class로 모든 Class를 객체화 할 순 없지만 간단한 데이터 전달을 위한 용도로 아주 편리하게 이용될 수 있다.
- Bundle 객체는 내부적으로 ArrayMap Collection을 사용한다.
    - ArrayMap을 사용하기 때문에 Parcelable 객체와 같이 데이터 순서에 맞춰 분해/조립을 하지 않아도 된다.
- Bundle을 이용하면 송수신 측에서 같은 Package안에 같은 Class를 공유하지 않더라도 data 를 전달시킬 수 있다. 
- Bundle은 Primitive Tpye 자료형을 전송하는데 아주 편리한 구조 이다.
    - Primitive Type : Boolean, Byte, Char, Short, int, Long, Float, Double, String, Parcelable, 각 Type Arry 등


- Bundle을 통해 intent에 data를 담아 보내는 예
    ```java
    private void SendDataViaBundle() {
        Bundle  bundle = new Bundle();

        String BundleDataKey1 = "BundleDataKey1";
        String BundleDataKey2 = "BundleDataKey2";
        String BundleDataKey3 = "BundleDataKey3";

        boolean	boolBundleData1 = true;
        String 	stringBundleData2 = "string data";
        int		intBundleData3 = 777;

        bundle.putBoolean(BundleDataKey1, boolBundleData1);
        bundle.putString(BundleDataKey2, stringBundleData2);
        bundle.putInt(BundleDataKey3, intBundleData3);

        Intent intent = new Intent();
        ComponentName cn =
                new ComponentName(m_packageName,
                        m_classNameBundle);
        intent.setComponent(cn);

        intent.putExtra(m_bundle_key, bundle);

        startActivity(intent);
    }
    ```
    

- Bundle을 통해 intent에 담긴 data를 얻어오는 예
  ```java
  public class BundleTestActivity extends AppCompatActivity {

    private final String m_bundle_key = "BundleDataKey";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bundle_test);

        Intent intent = getIntent();
        Bundle bundle = intent.getBundleExtra(m_bundle_key);

        String BundleDataKey1 = "BundleDataKey1"; // boolean
        String BundleDataKey2 = "BundleDataKey2"; // String
        String BundleDataKey3 = "BundleDataKey3"; // int

        assert bundle != null;

        boolean data1 = bundle.getBoolean(BundleDataKey1);
        String data2 = bundle.getString(BundleDataKey2);
        int data3 = bundle.getInt(BundleDataKey3);

        TextView tv1 = (TextView) findViewById(R.id.text_bundle_data1);
        tv1.setText(new StringBuilder().append("data1 : ").append(data1));

        TextView tv2 = (TextView) findViewById(R.id.text_bundle_data2);
        tv2.setText(new StringBuilder().append("data2 : ").append(data2));

        TextView tv3 = (TextView) findViewById(R.id.text_bundle_data3);
        tv3.setText(new StringBuilder().append("data3 : ").append(data3));
    }
  }
  ```

- Bundle 사용 예2    
    ```java
        public enum Api {
            API_TEST_ENUM(0x1234),
        }

        public void testFunction(long buffer) {
            final String KEY_NAME = "IMKEY";
            Bundle bundle = new Bundle();
            long buffer = 123;      
            bundle.putLong(KEY_NAME, (long)buffer);

            sendMsg(Api.API_TEST_ENUM, bundle);
        }

        private boolean sendMsg(int what, int arg1, int arg2, Bundle bundle) {
            Message message = Message.obtain(null, what);
            message.arg1 = arg1;
            message.arg2 = arg2;
            message.replyTo = mInCommingMessenger;
            if (bundle != null) {
                message.setData(bundle);
            }            

            // Messenger를 통한 Messager 전달 등 코드 Cont'd
        }
    ```