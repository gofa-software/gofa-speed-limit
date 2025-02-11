# Gofa speed limit library for android device

## Installation

Add it in your root build.gradle at the end of repositories:

- Groovy DSL

```groovy
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```

- Kotlin DSL

```kotlin
allprojects {
    repositories {
        ...
        maven(url = "https://jitpack.io")
    }
}
```

Add the dependency

- Groovy DSL

```groovy
dependencies {
    implementation 'com.github.gofa-software:gofa-speed-limit:1.0.7'
}
```

- Kotlin DSL

```kotlin
dependencies {
    implementation("com.github.gofa-software:gofa-speed-limit:1.0.7")

    val retrofitVersion = "2.9.0"
    implementation("com.squareup.retrofit2:retrofit:$retrofitVersion")
    implementation("com.squareup.retrofit2:converter-gson:$retrofitVersion")

    implementation("com.google.code.gson:gson:2.10.1")

    val coroutineVersion = "1.7.3"
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutineVersion")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutineVersion")

    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0")
}
```

## Usages
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(binding.root)

    GofaSpeedLimit.instance.setGofaKey("YOUR_GOFA_KEY")

    // 10 km/h
    GofaSpeedLimit.instance.setGpsBounds(10)
    // 100 m
    GofaSpeedLimit.instance.setDistanceShowSearchTrafficSign(100)

    GofaSpeedLimit.instance.limitSpeed.observe(this) {
        Log.d("MainActivity", "Speed limit: $it")
        binding.maxSpeed.text = it.toString()
    }

    GofaSpeedLimit.instance.error.observe(this) {
        Log.d("MainActivity", "OnError: $it")
    }

    GofaSpeedLimit.instance.trafficPoints.observe(this) {
        Log.d("MainActivity", "Traffic points: $it")
        binding.trafficSign.text = ""
        var title = ""
        it.forEach { trafficPoint ->
            if (trafficPoint.category == TrafficSignCategory.CAMERA) {
                if (trafficPoint.data?.isNotEmpty() == true) {
                    title += "${if (title.isNotEmpty()) "\n" else ""}Camera (${trafficPoint.data!!.first().distance?.toInt()}m)"
                }
            } else if (trafficPoint.category == TrafficSignCategory.TRAFFIC_SIGN) {
                trafficPoint.data?.forEach { point ->
                    val trafficBoard = when (point.instruction.toString()) {
                        TrafficSignInstruction.ENTER_URBAN.instruction -> "Vào khu dân cư"
                        TrafficSignInstruction.EXIT_TOWN.instruction -> "Thoát khỏi khu dân cư"
                        TrafficSignInstruction.START_NO_OVERTAKING.instruction -> "Đoạn đường cấm vượt"
                        TrafficSignInstruction.END_OVERTAKING.instruction -> "Hết cấm vượt"
                        else -> null
                    }

                    trafficBoard?.let {
                        title += "${if (title.isNotEmpty()) "\n" else ""}$trafficBoard (${point.distance?.toInt()}m)"
                    }
                }
            }
        }

        binding.trafficSign.text = title
    }

    GofaSpeedLimit.instance.speed.observe(this) {
        Log.d("MainActivity", "Speed: $it")
        binding.speed.text = it.toString()
    }
}
```

call update location
```kotlin
private fun onLocationChange(location: Location) {
    GofaSpeedLimit.instance.updateLocation(location)
}
```
