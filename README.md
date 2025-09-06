# VerticalIntSlider
Android Vertical Int Slider (Java) — custom View, no Material/SeekBar, integer snap, start/drag/end callbacks.

Here’s a polished, GitHub-ready example you can drop into a README. It’s fully in English, looks professional, and shows a clean way to use a vertical integer slider (since Google’s standard `Slider` doesn’t do vertical out of the box).

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
class VerticalIntSlider : View {
    var value: Int
    fun setRange(min: Int, max: Int)
    fun setOnValueChangeListener(l: OnValueChangeListener?)

    interface OnValueChangeListener {
        fun onStartTrackingTouch(view: VerticalIntSlider)
        fun onValueChanging(view: VerticalIntSlider, value: Int, fromUser: Boolean)
        fun onValueChangeFinished(view: VerticalIntSlider, value: Int)
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
