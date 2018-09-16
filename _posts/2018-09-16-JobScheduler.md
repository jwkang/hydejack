JobScheduler
===
## Job Scheduler 란 
연락처 동기화, 주기적으로 네트워크에 사용자 데이터를 업로드/다운로드 하는 경우 등과 같이 사용자에게 즉시 반영사항을 알릴 필요가 없는 작업들을 보통 백그라운드로 처리하게 된다.


# 사용 배경
## 왜 Android API로 존재할 까
Android 롤리팝(5.0) 버전 이후로 부터 jobSheduler가 지원된다. 
롤리팝 이전 App은 네트워크 요청사항이 있으면 즉시 처리해 버리는 구조로 돼 있었다. 모든 기능은 우선순 없이 자기 작업을 실행하게 되었고, 이경우 불필요한 베터리 소모 및 리소스 비효율성이 발생하였다. 

## Android Oreo 백그라운드 실행 제한과 위치 제한
Android 8.0는 사용자가 앱과 직접 상호작용하지 않을 때 이 앱이 수행할 수 있는 작업을 제한 한다

* 백그라운드 서비스 제한: 앱이 유휴 상태인 경우 백그라운드 서비스의 사용이 제한됩니다. 이 기능은 사용자에게 잘 보이는 포그라운드 서비스에는 적용되지 않습니다.
안드로이드 O에서는 앱이 백그라운드에 진입하게 되면 몇분 뒤 동작 중인 백그라운드 서비스는 자동으로 중지되며 `onDestroy()` 가 호출됩니다. 더하여 백그라운드 상태에서 서비스를 구동하기 위한 
`startService()`의 호출은 `IllegalStateException`이 발생하며 허용되지 않습니다.	   
이러한 백그라운드 실행 제한은 target SDK가 Android O(API Level 26) 이상인 경우에 적용됩니다. 
    * 1분만에 onDestory가 호출됨

* 브로드캐스트 제한: 제한된 예외의 경우, 앱이 암시적 브로드캐스트에 등록하기 위해 자체 매니페스트를 사용할 수 없습니다. 
그렇지만 여전히 앱이 런타임에 브로드캐스트에 등록할 수 있으며, 특정 앱을 대상으로 하는 명시적 브로드캐스트에 등록하기 위해 매니페스트를 사용할 수 있습니다.
(암시적(Implicit) 브로드캐스트 인텐트의 제한)

* 백그라운드 서비스는 아래 어떤 조건에도 해당하지 않는 서비시를 의미한다.
    * 액티비티가 시작되거나 일시 중지되거나 상관없이 보이는 액티비티가 있는 경우
    * 포그라운드 서비스가 있는 경우( `startForeground()`로 실행된 서비스)
    * 앱의 서비스 중 하나에 바인드하거나 앱의 콘텐츠 제공자 중 하나를 사용하여 앱에 또 다른 포그라운드 앱이 연결된 경우 예를 들어 다른 앱이 다음 항목에 바인드되어 있다면 포그라운드에 있는 것입니다.
        * `Context.startForegroundService()`로 실행해 위 제한을 회피할 수도 있다.

# JobScheduler 사용하기
## 대표적인 함수
1. `onStartJob()` : Job이 수행될 때 불리는 callback 함수.
   false return시 함수 종료 시점에 system에 작업이 끝남을 알림.
   true return시 함수가 종료됐더라도 작업이 끝나지 않은 상태로 둠. `jobFinished()`함수를 호출해야만 해당 job이 종료된다.    
2. `jobFinished()` : JobScheduler에 의해 생성된 background service 역시 일반 service와 마찬가지로 **main thread**에서 동작한다. `onStartJob()`가 true를 return해주는 경우 보통
   thread를 생성한 후 그 thread 함수에서 `jobFinished()` 함수를 호출시켜 job을 종료 시킨다. 2개의 인자를 가지고 있으며, 첫번째는 job의 id 두번 째는 retry여부이다.    
3. `onStop()` : Job이 **완료되기 전 취소**된 경우 호출되는 callback 함수 이다. 예를 들어 job을 plugin이 됐을 때를 condition으로 놓고 build했다면, 만약 그 condition이 false가 될 때 jobScheduler에 등독된 job은 취소되게 된다. true를 return하면 rescheduling 된다. 

## job 빌드 방법 및 condition 지정
* `jobScheulder`에 의해 시작될 background service의 경우 scheduler에 의해 binding돼 사용돼야 하므로 `AndroidMenifest`에 permmision이 등록돼야 한다.
```xml
       <service android:name=".jw_test_service"
            android:permission="android.permission.BIND_JOB_SERVICE"
            android:exported="true">
        </service>
```
* `JobScheduelr`는 기본적으로 시간을 기준으로 동작하는 것이 아니라 **조건을 기반으로 동작** 한다. 
* `JobInfo`를 통해 실행할 job의 조건을 등록할 수 있다.
* 조건으로 줄 수 있는 type은 다음과 같다. (TV input에서 주로 사용된 예를 바탕으로 설명)
    - `Persistent` : reboot이 되더라도 자동으로 jobSchedular에 등록. 되어야 한다면 사용하는 조건입니다.
    꼭 RECEIVE_BOOT_COMPLETED permission을 추가해야 함
    - `NetworkType` : 유료/무료(metered/unmetered) 네트워크 또는 연결 비연결 네트워크에서 동작하도록 구분지어 조건을 걸 수 있음
    - `Periodic` : 주기적으로 동작해야 한다면, 반복 시간을 등록할 수 있다. **주의할 점** 으로 Android N버전 이후 부터는 Periodic을 설정할 때 반드시 `FlexMillis` 값을 설정해야만 동작한다.
    > In Android Nougat the setPeriodic(long intervalMillis) method call makes use of setPeriodic (long intervalMillis, long flexMillis) to schedule periodic jobs.
    > The job can execute at any time in a window of flex length at the end of the period.
    - `Extras` : 추가 정보를 넘겨야 하는 경우 사용. Bundle을 넣어 사용하면 여러가지 type을 전달할 수 있다.     

```java
    PersistableBundle persistableBundle = new PersistableBundle();
            persistableBundle.putString(EpgSyncJobService.BUNDLE_KEY_INPUT_ID, inputId);
            persistableBundle.putLong(EpgSyncJobService.BUNDLE_KEY_SYNC_PERIOD, syncDuration);
            JobInfo.Builder builder = new JobInfo.Builder(PERIODIC_SYNC_JOB_ID, jobServiceComponent);

    JobInfo.Builder builder = new JobInfo.Builder(PERIODIC_SYNC_JOB_ID, jobServiceComponent);
            JobInfo jobInfo = builder
                    .setExtras(persistableBundle)
                    .setPeriodic(fullSyncPeriod)
                    .setPersisted(true)
                    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)
                    .build();

    scheduleJob(context, jobInfo);
```

```xml
    <!-- Required to sync EPG data after reboot. For details,
    please see {@link JobInfo.Builder#setPersisted}-->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

## JobScheduler 구성요소
1. `JobInfo` : Job이 실행된 조건을 설정하고, `JobScheduler`를 통해 이를 시스템에 등록함
2. `JobService` : Job 실행시 필요한 동작을 구현
3. `android.permission.BIND_JOB_SERVICE` permission : Job 실행을 위한 권한

## JobIntentService 사용하기
