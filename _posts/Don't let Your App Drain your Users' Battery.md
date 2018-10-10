---
title: Don't let Your App Drain your Users' Battery 
tags: [Android]
---

### What drives battery Life?

1. Hardware (Screen etc.)
2. Operating System
3. Apps & Services
4. User Interaction

### Efforts to improve battery

1. [Job Scheduler](https://developer.android.google.cn/reference/android/app/job/JobScheduler) (Since API 21)
2. Doze & App Standby(Since API 23)
3. Doze on-the-go (Since API 24)
4. Background Limits (Since API 26)
5. Adaptive Battery, Background Restrictions etc. (Since API 28)

Among the above **only Job Scheduler** could be directly leveraged by app developers in the code.

#### Job Scheduler (Since API 21)

Job Scheduler is suitable when we want to do something in a specified circumstance, such as:

> Batterry is not Low
> 
> With specified Netwoek Status
> 
> Is Charging
> 
> Storage is not Low
> 
> Device is Idle

Without Job Scheduler, we may need to keep a service running to monitor specified system broadcast and then do what we want, which is not optimized for battery comparing to Job Scheduler. 

Here is a simple example of Job Scheduler:

```java
public class MyJobService extends JobService {
    @Override
    public boolean onStartJob(final JobParameters params) {

        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... voids) {
                // stuffs that consumes a lot of time like making a backup to the cloud
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                // tell scheduler our job is done
                jobFinished(params, false);
            }
        }.execute();

        // return true to tell scheduler our job is not finished.
        return true;
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        return false;
    }
}

public class MainActivity extends Activity {

    private static int sJobId = 0;
    private JobScheduler mJobScheduler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mJobScheduler = (JobScheduler)getSystemService(Context.JOB_SCHEDULER_SERVICE);

        findViewById(R.id.schedule_btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                scheduleAJob();
            }
        });
    }

    public void scheduleAJob() {
        for (JobInfo info : mJobScheduler.getAllPendingJobs()) {
            if (info.getId() == sJobId) {
                // the last scheduled job is not finished yet
                return;
            }
        }

        // build the jobInfo that requires charging, idle and network that won't charge.
        JobInfo jobInfo = new JobInfo.Builder(++ sJobId, new ComponentName(this, MyJobService.class))
                .setRequiresCharging(true)
                .setRequiresDeviceIdle(true)
                .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
                .build();

        mJobScheduler.schedule(jobInfo);
    }
}
```

#### Doze (Since API 23)

Unplugged and stationary for a period of time, a device would be in doze mode, which restricts all apps regardless whether they targets api 23. However, the OS periodically exits Doze for a brief time to let apps complete their deferred jobs. The bried time slot is called Maintenance window. Below is a figure showing the mode changing:

![](http://tao93.top/images/2018/10/10/1539151703.png)

In doze mode, the following are restricted:

1. Network access is suspended
2. WakeLock is ignored
3. Standard AlarmManager alarms (setExact() and setWindow()) are deferred to the next maintenance window
4. No Wi-Fi scaning
5. No [Sync Adapters](https://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html)
6. No JobScheduler jobs

Stuffs not restricted by Doze:

1. FCM high priority msg
2. Alarms set with setAndAllowWhileIdle() and setExactAndAllowWhileIdle()

Testing Doze:

```bash
# if your device is connected with a cable, use the following to disable charging
adb shell dumpsys battery unplug

# Force the system into Doze mode
adb shell dumpsys deviceidle force-idle

# exit idle mode & reactivate the device
adb shell dumpsys deviceidle unforce
adb shell dumpsys battery reset # this also recovers charging
```

#### App Standby (Since API 23)

An app is idle if the followings are satisfies:

1. No user touching for a certain period of time
2. No processes in the foreground (either as an activity or foreground service)
3. No notifications showing on the lock screen or in the notification center.
4. Not charging

Testing App Standby:

```bash
# if your device is connected with a cable, use the following to disable charging
adb shell dumpsys battery unplug

# check whether your app is in standby mode. A 'Idle=false' output means not in standby mode
adb shell am get-inactive <packageName>

# Force the app into App Standby mode
adb shell am set-inactive <packageName> true

# exit Standby mode for your app
adb shell am set-inactive <packageName> false
```

### Doze on-the-go (Since API 24)

A lighter Doze mode which activates when the phone is moving in our pockets or hands. Doze on-the-go allows WakeLock, Wifi Scaning and GPS etc, that's why it's lighter than former Doze mode introduced in API 23.

Ignore Doze configuration:

![](http://tao93.top/images/2018/10/10/1539154768.png)

#### Background Limits (Since API 26)

Background Limits affects apps that targets API 26 or higher and includes Background Service Limitations and Broadcast Limitations.

##### Background Service Limitations

For an app that targets API 26 or higher, it's in background if:

1. No visible Activity
2. No foreground Service
3. Not InputMethod Service, Wallpaper service etc.

After several minutes of being in background, background services are stopped by the OS. Replacing background services with Scheduler Jobs is fine in many cases.

##### Broadcast Limitations (Introduced in API 25 and strengthened in API 26)

For an app that targets API 26 or higher, it:

1. can't register receivers for implicit broadcasts in Manifest file
2. can register receivers for explicit broadcasts in Manifest file
3. can register receivers for any broadcasts runtimely

Broadcasts that require a [signature permission](https://developer.android.com/guide/topics/manifest/permission-element.html#plevel) are exempted from this restriction.

In some cases, registering system broadcasts could be replaced by Scheduler Jobs, such as if we want to do something when the device is charging.

### Adaptive Battery (Since API 29)

A new feature based on Machine Learning.

1. Limit battery for apps that are not used often.
2. Apps should be able to run quickly when they are needed.
3. Don't bother users to manage manually.

Apps are arranged into 4 standby buckets: `Active`, `Working set`, `Frequent` and `Rare`. Limits are increased from `Active` to `Rare`:

![](http://tao93.top/images/2018/10/10/1539157914.png)

An app is in `Active` if it:

1. has launched an activity
2. is running a foreground service
3. has a sync adapter associated with a content provider used by a foreground app
4. has a notification clicked by the user

An app is in `Working set` if it runs often but isn't active.

An app is in `Frequent` if it is used regularly, but not necessarily every day.

An app is in `Rare` if it is not often used.

Find out what bucket the app is currently in programmatically:

```java
UsageStatsManager.getAppStandbyBucket()
```

Test Standby Buckets:

```bash
# assgin your app into a specified bucket
adb shell am set-standby-bucket <packagename> active|working_set|frequent|rare

# check bucket assignments for one app or all apps
# the output '10 20 30 40' mean Active, Working set, Frequent, Rare respectively
adb shell am get-standby-bucket [<packagename>]
```

### Battery Saver

New battery saver in API 29:

1. No red status bar and has animation
2. Location service is off when screen is off
3. Battery level threshold is adjustable

Apps are encouraged to switch to dark theme when battery saver is on.

Check whether battery saver is on programmatically:

```java
((PowerManager)getSystemService(Context.POWER_SERVICE)).isPowerSaveMode()
```

Test battery saver:

```bash
# pretend to be in low battery status
adb shell settings put global low_power 1

# reset all configurations
adb shell dumpsys battery reset
```

### Background Restrictions (Since API 29)

Two criterias:

1. Apps targeting pre-Oreo and using background services
2. Excessive WakeLocks (> 1hr in background)

Background restrictions are decided by the users:

![](http://tao93.top/images/2018/10/10/1539156945.png)

When Background Restrictions is enabled for an app, fllowings are restricted:

1. Background jobs, alarms, services and network accessing
2. Location related updates 
3. Foreground services

Except GUI operation, Background restrictions could alse be finished via adb:

```bash
# enable background restrictions
adb shell appops set <package_name> RUN_ANY_IN_BACKGROUND ignore

# disable background restrictions
adb shell appops set <package_name> RUN_ANY_IN_BACKGROUND allow
```

