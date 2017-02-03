---
layout: post
comments: false
title: LinearTimer v2.0.0
---

Linear Timer is a custom view for Android that enables circular progress animation with respect to given duration.

[Find LinearTimer on Github](https://github.com/krtkush/LinearTimer)

## Usage

First, you need to add `LinearTimerView` into your XML layout -

    xmlns:app="http://schemas.android.com/apk/res-auto"

    <io.github.krtkush.lineartimer.LinearTimerView
        android:id="@+id/linearTimer"
        android:layout_centerHorizontal="true"
        android:layout_width="120dp"
        android:layout_height="120dp"
        app:radius="20dp"
        app:strokeWidth="3dp"/>

List of attributes available to toggle the LinearTimer's basic style -

1. **radius** - The radius of the circle.
2. **strokeWidth** - the thickness of the circle boundary.
3. **startingPoint** - The angle from where, in the timer, you want the animation to start. 270 is the 12 O'Clock position.
4. **initialColor** - The initial color of the circle.
5. **progressColor** - The color of the progress arc that animates over the initial color.


After adding the view, here is how the View is initialized and used -

     LinearTimerView linearTimerView = (LinearTimerView)
                                        findViewById(R.id.linearTimer);

     LinearTimer linearTimer = new new LinearTimer.Builder()
                .linearTimerView(linearTimerView)
                .duration(10 * 1000)
                .build();

    // Start the timer.
    findViewById(R.id.startTimer).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
          linearTimer.startTimer();
        }
    });

    // Restart the timer.
    findViewById(R.id.restartTimer).setOnClickListener(new View.OnClickListener() {
         @Override
         public void onClick(View view) {
            linearTimer.restartTimer();
          }
    });

**Duration**

The `duration()` method is overloaded for the following two cases -

1. The user provides the total duration and the timer runs for that given duration.
2. The user provides the total duration and the elapsed time duration. In this case, LinearTimer will automatically calculate the pre-fill angle according to the time elapsed. Also, any value passed into `preFillAngle()` method will be ignored.

`new LinearTimer.Builder().duration(10 * 1000).build();` or `new LinearTimer.Builder().duration(10 * 1000, 5 * 1000).build();`

**Timer Listener**

A `TimerListener` can be implemented to receive call backs for various LinearTimer related events.

1. animationComplete() - When the progress animation comes to an end.
2. timerTick(long tickUpdateInMillis) - If the user chooses to receive the time elapsed since the timer has started or the time remaining for the timer to finish.  

Implement the following interface in your activity -

`...implements LinearTimer.TimerListener`

Make sure you pass the listener in the LinearTimer builder -

`...new LinearTimer.Builder().timerListener(this).build();`

And then, override the `animationComplete` and `timerTick` methods.

    @Override
    public void animationComplete() {
      Log.i("Animation", "complete");
    }

    @Override
    public void timerTick(long tickUpdateInMillis) {
        Log.i("Time left or Time elapsed", String.valueOf(tickUpdateInMillis));
    }

**Progress Direction**

The user can choose to make the progress bar animate in either clockwise or anti-clockwise manner. To change the direction `progressDirection(int progressDirection)` method needs to be added while building LinearTimer.

`...new LinearTimer.Builder().progressDirection(LinearTimer.COUNTER_CLOCK_WISE_PROGRESSION).build();`

progressDirection() can have either of the following two parameters -

1. LinearTimer.COUNTER_CLOCK_WISE_PROGRESSION
2. LinearTimer.CLOCK_WISE_PROGRESSION


The default direction is set as clockwise.

**Pre-fill Angle**

If the user wants to pre-fill the circle with the progress color `preFillAngle(float angle)` needs to be added while building LinearTimer. The argument contains the angle till which the pre-fill should occur.

`...new LinearTimer.Builder().preFillAngle(10).build();`

The default value is 0.

**Ending Angle**

User can change the angle at which the progress ends by adding `endingAngle(int endingAngle)` method to the Builder.

`...new LinearTimer.Builder().endingAngle(360).build();`

The default value is 0.

**Count Updates**

The user can choose to receive the time elapsed since the timer has started or the time remaining for the timer to finish. `getCountUpdate(int countType, long updateInterval)` needs to be added to the builder along with implementation of `TimerListener`.

`...new LinearTimer.Builder().getCountUpdate(LinearTimer.COUNT_UP_TIMER, 1000).build();`

`countType` can be of the following two types -

1. LinearTimer.COUNT_UP_TIMER
2. LinearTimer.COUNT_DOWN_TIMER

`updateInterval` is the duration after which `timerTick()` should get an update. This value should always be > 0.
