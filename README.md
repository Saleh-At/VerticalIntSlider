# VerticalIntSlider
A clean way to use a vertical integer slider (since Google’s standard `Slider` doesn’t do vertical out of the box).

---

# VerticalIntSlider — Android (Vertical Integer Slider)

A lightweight, highly-customizable **vertical** slider for Android that snaps to **integer** steps. Perfect for volume, brightness, zoom, or any range-based control where vertical UX just makes more sense.

> ✅ Zero dependencies · 🎯 Integer steps · 🎨 Fully themeable · ♿ Accessible · 📳 Haptics support

---

## Why this exists

Android’s Material `Slider` is horizontal by default and doesn’t expose a simple, compact **vertical** integer slider. `VerticalIntSlider` fills that gap with a small custom `View` designed for production apps.

---

## Features

* **Vertical** orientation with smooth touch tracking
* **Integer** step values between `vis_min` and `vis_max`
* Customizable **track**, **progress**, **thumb**, **text** colors
* Adjustable **track width** and **thumb radius**
* Optional **ticks** and **numbers**
* **Haptics** on value commit (API-aware)
* **Accessibility** with content descriptions and announcements
* **State saving** across configuration changes

---

## XML attrs

Declare these once (usually in `res/values/attrs.xml`):

```xml
<resources>
    <declare-styleable name="VerticalIntSlider">
        <attr name="vis_min" format="integer" />
        <attr name="vis_max" format="integer" />
        <attr name="vis_trackColor" format="color" />
        <attr name="vis_progressColor" format="color" />
        <attr name="vis_thumbColor" format="color" />
        <attr name="vis_textColor" format="color" />
        <attr name="vis_trackWidth" format="dimension" />
        <attr name="vis_thumbRadius" format="dimension" />
        <attr name="vis_showTicks" format="boolean" />
        <attr name="vis_showNumbers" format="boolean" />
    </declare-styleable>
</resources>
```

---

## Quick start (XML)

```xml
<com.yourpkg.widgets.VerticalIntSlider
    android:id="@+id/rightSlider"
    android:layout_width="@dimen/_35sdp"
    android:layout_height="0dp"
    android:padding="@dimen/_14sdp"
    app:layout_constraintHeight_percent="0.45"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintBottom_toTopOf="@id/lowVolumeRightIcon"

    app:vis_min="1"
    app:vis_max="10"
    app:vis_trackColor="@color/sliderColor"
    app:vis_progressColor="@color/activeTabColor"
    app:vis_thumbColor="@color/activeTabColor"
    app:vis_textColor="@android:color/white"
    app:vis_trackWidth="@dimen/_4sdp"
    app:vis_thumbRadius="@dimen/_14sdp"
    app:vis_showTicks="true"
    app:vis_showNumbers="false" />
```

---

## Listening to value changes (Kotlin)

This improves on your snippet with (1) safe haptics on commit, (2) UI updates during drag, and (3) accessibility announcements:

```kotlin
binding.rightSlider.setOnValueChangeListener(object : VerticalIntSlider.OnValueChangeListener {
    override fun onStartTrackingTouch(view: VerticalIntSlider) {
        // Optional: highlight or show a tooltip
    }

    override fun onValueChanging(view: VerticalIntSlider, value: Int, fromUser: Boolean) {
        if (!fromUser) return
        binding.rightSliderText.text = value.toString()
        // Optional: live-preview something (e.g., audio volume)
    }

    override fun onValueChangeFinished(view: VerticalIntSlider, value: Int) {
        // Haptics (API-aware)
        val vibrator = view.context.getSystemService(android.os.Vibrator::class.java)
        if (vibrator != null) {
            if (android.os.Build.VERSION.SDK_INT >= 26) {
                vibrator.vibrate(
                    android.os.VibrationEffect.createOneShot(35, android.os.VibrationEffect.DEFAULT_AMPLITUDE)
                )
            } else {
                @Suppress("DEPRECATION")
                vibrator.vibrate(35)
            }
        }

        // Accessibility announcement
        view.announceForAccessibility("Value set to $value")

        // Apply the final value to your app logic
        // DeviceManager.getInstance().setVolumeRight(value)
    }
})
```

### Programmatic control

```kotlin
binding.rightSlider.value = 7        // set
val current = binding.rightSlider.value
binding.rightSlider.setRange(1, 20)  // change min/max at runtime
```

---

## Best practices baked in

* **Smooth interaction**: live updates during drag (`onValueChanging`), commit work at release (`onValueChangeFinished`)
* **Haptics**: short, subtle feedback using `VibrationEffect` on API 26+ with a safe fallback
* **A11y**: content descriptions + announcements so TalkBack users aren’t left behind
* **State**: `onSaveInstanceState` / `onRestoreInstanceState` inside the view to survive rotations

---

## Minimal Java usage (if you prefer Java)

```java
rightSlider.setOnValueChangeListener(new VerticalIntSlider.OnValueChangeListener() {
    @Override public void onStartTrackingTouch(VerticalIntSlider view) { }

    @Override public void onValueChanging(VerticalIntSlider view, int value, boolean fromUser) {
        if (!fromUser) return;
        rightSliderText.setText(String.valueOf(value));
    }

    @Override public void onValueChangeFinished(VerticalIntSlider view, int value) {
        android.os.Vibrator vib = view.getContext().getSystemService(android.os.Vibrator.class);
        if (vib != null) {
            if (android.os.Build.VERSION.SDK_INT >= 26) {
                vib.vibrate(android.os.VibrationEffect.createOneShot(35,
                        android.os.VibrationEffect.DEFAULT_AMPLITUDE));
            } else {
                //noinspection deprecation
                vib.vibrate(35);
            }
        }
        view.announceForAccessibility("Value set to " + value);
        // DeviceManager.getInstance().setVolumeRight(value);
    }
});
```

---

## Styling tips

* Keep `trackWidth` between `2dp`–`6dp` for a balanced look
* Use a **high-contrast** `thumbColor` vs `trackColor` for visibility
* If you enable `vis_showNumbers`, keep `textColor` readable against your background
* For compact UIs, hide numbers (`vis_showNumbers="false"`) and rely on a label

---

## Public API (suggested)

```kotlin
package com.yourpkg.widgets;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.os.Parcel;
import android.os.Parcelable;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;

import androidx.annotation.Nullable;

import com.yourpkg.R;

/**
 * A custom vertical slider that works with integer values.
 * <p>
 * Features:
 * - Integer step values between min/max
 * - Customizable colors (track, progress, thumb, text)
 * - Adjustable track width and thumb radius
 * - Optional ticks and numbers
 * - Notifies listener during drag and on release
 *
 * Designed for use cases where Android’s default horizontal Slider
 * is not a good fit (volume, brightness, zoom, etc).
 */
public class VerticalIntSlider extends View {

    // Attributes
    private int minValue = 0;
    private int maxValue = 10;
    private int value = 0;

    private int trackColor = Color.GRAY;
    private int progressColor = Color.BLUE;
    private int thumbColor = Color.RED;
    private int textColor = Color.BLACK;

    private float trackWidth = 4f;
    private float thumbRadius = 20f;

    private boolean showTicks = false;
    private boolean showNumbers = false;

    // Drawing
    private Paint trackPaint;
    private Paint progressPaint;
    private Paint thumbPaint;
    private Paint textPaint;

    // Listener
    private OnValueChangeListener listener;

    public VerticalIntSlider(Context context) {
        super(context);
        init(null);
    }

    public VerticalIntSlider(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init(attrs);
    }

    public VerticalIntSlider(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(attrs);
    }

    private void init(@Nullable AttributeSet attrs) {
        trackPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        progressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        thumbPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setTextAlign(Paint.Align.CENTER);
        textPaint.setTextSize(36f);

        if (attrs != null) {
            TypedArray a = getContext().obtainStyledAttributes(attrs, R.styleable.VerticalIntSlider);

            minValue = a.getInt(R.styleable.VerticalIntSlider_vis_min, minValue);
            maxValue = a.getInt(R.styleable.VerticalIntSlider_vis_max, maxValue);
            trackColor = a.getColor(R.styleable.VerticalIntSlider_vis_trackColor, trackColor);
            progressColor = a.getColor(R.styleable.VerticalIntSlider_vis_progressColor, progressColor);
            thumbColor = a.getColor(R.styleable.VerticalIntSlider_vis_thumbColor, thumbColor);
            textColor = a.getColor(R.styleable.VerticalIntSlider_vis_textColor, textColor);
            trackWidth = a.getDimension(R.styleable.VerticalIntSlider_vis_trackWidth, trackWidth);
            thumbRadius = a.getDimension(R.styleable.VerticalIntSlider_vis_thumbRadius, thumbRadius);
            showTicks = a.getBoolean(R.styleable.VerticalIntSlider_vis_showTicks, showTicks);
            showNumbers = a.getBoolean(R.styleable.VerticalIntSlider_vis_showNumbers, showNumbers);

            a.recycle();
        }

        updatePaints();
    }

    private void updatePaints() {
        trackPaint.setColor(trackColor);
        trackPaint.setStrokeWidth(trackWidth);

        progressPaint.setColor(progressColor);
        progressPaint.setStrokeWidth(trackWidth);

        thumbPaint.setColor(thumbColor);

        textPaint.setColor(textColor);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        int width = getWidth();
        int height = getHeight();
        int centerX = width / 2;

        // Draw track
        canvas.drawLine(centerX, thumbRadius, centerX, height - thumbRadius, trackPaint);

        // Draw progress
        float progressRatio = (float) (value - minValue) / (maxValue - minValue);
        float thumbY = height - thumbRadius - progressRatio * (height - 2 * thumbRadius);
        canvas.drawLine(centerX, height - thumbRadius, centerX, thumbY, progressPaint);

        // Draw ticks
        if (showTicks) {
            int steps = maxValue - minValue;
            for (int i = 0; i <= steps; i++) {
                float y = height - thumbRadius - i * (height - 2 * thumbRadius) / steps;
                canvas.drawLine(centerX - 20, y, centerX + 20, y, trackPaint);
            }
        }

        // Draw numbers
        if (showNumbers) {
            int steps = maxValue - minValue;
            for (int i = 0; i <= steps; i++) {
                int num = minValue + i;
                float y = height - thumbRadius - i * (height - 2 * thumbRadius) / steps;
                canvas.drawText(String.valueOf(num), centerX - 40, y + 12, textPaint);
            }
        }

        // Draw thumb
        canvas.drawCircle(centerX, thumbY, thumbRadius, thumbPaint);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int height = getHeight();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                if (listener != null) {
                    listener.onStartTrackingTouch(this);
                }
                updateValueFromTouch(event.getY(), true, false);
                return true;

            case MotionEvent.ACTION_MOVE:
                updateValueFromTouch(event.getY(), true, true);
                return true;

            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                updateValueFromTouch(event.getY(), true, false);
                if (listener != null) {
                    listener.onValueChangeFinished(this, value);
                }
                return true;
        }

        return super.onTouchEvent(event);
    }

    private void updateValueFromTouch(float y, boolean fromUser, boolean isDragging) {
        int height = getHeight();
        float ratio = (height - thumbRadius - y) / (height - 2 * thumbRadius);
        ratio = Math.max(0f, Math.min(1f, ratio));

        int newValue = minValue + Math.round(ratio * (maxValue - minValue));
        if (newValue != value) {
            value = newValue;
            invalidate();
            if (listener != null) {
                listener.onValueChanging(this, value, fromUser);
            }
        }
    }

    // Public API

    public void setOnValueChangeListener(OnValueChangeListener listener) {
        this.listener = listener;
    }

    public void setRange(int min, int max) {
        this.minValue = min;
        this.maxValue = max;
        if (value < minValue) value = minValue;
        if (value > maxValue) value = maxValue;
        invalidate();
    }

    public int getValue() {
        return value;
    }

    public void setValue(int v) {
        if (v < minValue) v = minValue;
        if (v > maxValue) v = maxValue;
        if (this.value != v) {
            this.value = v;
            invalidate();
        }
    }

    // State saving (e.g. on config change)
    @Override
    protected Parcelable onSaveInstanceState() {
        Parcelable superState = super.onSaveInstanceState();
        SavedState ss = new SavedState(superState);
        ss.value = this.value;
        return ss;
    }

    @Override
    protected void onRestoreInstanceState(Parcelable state) {
        if (!(state instanceof SavedState)) {
            super.onRestoreInstanceState(state);
            return;
        }
        SavedState ss = (SavedState) state;
        super.onRestoreInstanceState(ss.getSuperState());
        this.value = ss.value;
    }

    static class SavedState extends BaseSavedState {
        int value;

        SavedState(Parcelable superState) {
            super(superState);
        }

        private SavedState(Parcel in) {
            super(in);
            this.value = in.readInt();
        }

        @Override
        public void writeToParcel(Parcel out, int flags) {
            super.writeToParcel(out, flags);
            out.writeInt(this.value);
        }

        public static final Creator<SavedState> CREATOR =
                new Creator<SavedState>() {
                    @Override
                    public SavedState createFromParcel(Parcel in) {
                        return new SavedState(in);
                    }

                    @Override
                    public SavedState[] newArray(int size) {
                        return new SavedState[size];
                    }
                };
    }

    // Listener interface
    public interface OnValueChangeListener {
        void onStartTrackingTouch(VerticalIntSlider view);
        void onValueChanging(VerticalIntSlider view, int value, boolean fromUser);
        void onValueChangeFinished(VerticalIntSlider view, int value);
    }
}

```

---

## Troubleshooting

* **Thumb not centered?** Ensure the view has enough horizontal width (e.g., `>= 28dp + padding`)
* **Not receiving callbacks?** Verify your listener is set on the same instance referenced in XML
* **Clamped values?** Values outside `[vis_min, vis_max]` are clamped; double-check your range

---

## Contributing

Issues and PRs are welcome! Add tests for touch edge cases (top/bottom clamp, rapid drags), and consider a sample app module with a few presets:

* Volume (1–10, no numbers)
* Zoom (10–100, ticks every 10)
* Temperature (16–30, numbers on)
