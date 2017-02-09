---
layout: post
comments: true
title: Android - Linear Circular Progress Bar
tags: [Code, LinearTimer, Android]
---

I'm planning to write an open source library for a countdown/ up timer with circular
progress animation around it. Before I do that I have decided to write a post about
the basics of making a progress animation directly associated with time give.

<img src="https://raw.githubusercontent.com/krtkush/krtkush.github.io/master/_posts/images/circular_progress_bar_screenshot.png" width="250" height="350" />

We'll need a custom view class and an animation class that'll control the animation.


**Custom View**

      public class TimerCircle extends View {

        private Paint paint;
        private RectF rectF;

        // The point from where the color-fill animation will start.
        private static final int startingPointInDegrees = 270;
        // The point up-till which user wants the circle to be pre-filled.
        private float degreesUpTillPreFill = 0;

        public TimerCircle(Context context, AttributeSet attrs) {
            super(context, attrs);

            int strokeWidth = (int) convertDpIntoPixel(5);

            paint = new Paint();
            paint.setAntiAlias(true);
            paint.setStyle(Paint.Style.STROKE);
            paint.setStrokeWidth(strokeWidth);

            // Define the size of the circle
            rectF = new RectF(strokeWidth, strokeWidth,
                  (int) convertDpIntoPixel(100) + strokeWidth,
                  (int) convertDpIntoPixel(100) + strokeWidth);
        }          

        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);

            // Grey Circle - This circle will be there by default.
            paint.setColor(Color.parseColor("#d2d2d2"));
            canvas.drawCircle(rectF.centerX(), rectF.centerY(),
                    (int) convertDpIntoPixel(50), paint);

            // Green Arc (Arc with max 360 angle) - This circle will be created as
            // time progresses.
            paint.setColor(Color.parseColor("#00ba8c"));
            canvas.drawArc(rectF, startingPointInDegrees, degreesUpTillPreFill,
               false, paint);
        }

        public float getDegreesUpTillPreFill() {
            return degreesUpTillPreFill;
        }

        public void setDegreesUpTillPreFill(float degreesUpTillPreFill) {
            this.degreesUpTillPreFill = degreesUpTillPreFill;
        }

        // Method to convert DPs into Pixels.
        private float convertDpIntoPixel(float dp) {
            float scale = getResources().getDisplayMetrics().density;
            return dp * scale + 0.5f;
        }
      }


**Animation**

      public class TimerCircleAngleAnimation extends Animation {

          private TimerCircle circle;

          private float startingAngle;
          private float endingAngle;

          public TimerCircleAngleAnimation(TimerCircle circle, int endingAngle) {
              this.startingAngle = circle.getDegreesUpTillPreFill();
              this.endingAngle = endingAngle;
              this.circle = circle;
          }

          @Override
          protected void applyTransformation(float interpolatedTime,
            Transformation transformation) {

              float finalAngle = startingAngle + ((endingAngle - startingAngle)
                    * interpolatedTime);

              circle.setDegreesUpTillPreFill(finalAngle);
              circle.requestLayout();
          }
      }


**Usage**

1. XML -

        <io.github.krtkush.timer.TimerCircle
          android:id="@+id/timerCircle"
          android:layout_width="110dp"
          android:layout_height="110dp"
          android:background="@null"
          android:layout_centerInParent="true"/>

2. Java -

        TimerCircle centerTimerCircle = (TimerCircle) findViewById(R.id.timerCircle);

        centerTimerCircle.setDegreesUpTillPreFill(0);
        // The arc will be of 360 degrees - a circle.
        TimerCircleAngleAnimation animation =
            new TimerCircleAngleAnimation(centerTimerCircle, 360);
        // Timer will run for 1 minute.
        animation.setDuration((60 * 1000));
        centerTimerCircle.startAnimation(animation);
