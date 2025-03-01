# APK Build Organizer  

A small Gradle task that **renames APKs** with useful details and **organizes them into a separate folder**.  
No more manually renaming files—just build your APK, and this task will handle the rest! 🚀  

## 🔹 What It Does  
✅ **Automatically renames APKs** after every build  
✅ **Includes** package name, Git branch, build type, version, and timestamp in the filename  
✅ **Keeps the original APK** in place and moves the renamed file to a `modified` folder  
✅ **Cleans up old APKs** before adding new ones  

---

## 📌 Installation & Setup  

### 1️⃣ Add the Task to `build.gradle.kts`  
Copy and paste this **inside your `build.gradle.kts (Module: app)`** file:  

```kotlin
import java.text.SimpleDateFormat
import java.util.*
import java.io.ByteArrayOutputStream

tasks.register("moveAndRenameApk") {
    group = "custom"
    description = "Deletes old files in the 'modified' folder and renames APKs with Git branch info and timestamp."

    doLast {
        val buildDirFile = layout.buildDirectory.get().asFile
        val outputDirs = listOf("debug", "release")

        val gitBranch: String = try {
            val stdout = ByteArrayOutputStream()
            exec {
                commandLine("git", "rev-parse", "--abbrev-ref", "HEAD")
                standardOutput = stdout
            }
            stdout.toString().trim().replace("/", "_")
        } catch (e: Exception) {
            println("Failed to retrieve Git branch: ${e.message}")
            "unknown"
        }

        println("Git branch: $gitBranch")

        outputDirs.forEach { type ->
            val apkDir = file("\${buildDirFile}/outputs/apk/\$type")
            if (apkDir.exists()) {
                val modifiedDir = File(apkDir, "modified")
                if (modifiedDir.exists()) {
                    modifiedDir.listFiles()?.forEach { file -> if (file.isFile) file.delete() }
                } else {
                    modifiedDir.mkdirs()
                }

                apkDir.listFiles { file -> file.name.endsWith(".apk") }?.forEach { apkFile ->
                    val packageName = android.defaultConfig.applicationId?.replace(".", "_")
                    val versionName = android.defaultConfig.versionName
                    val versionCode = android.defaultConfig.versionCode
                    val dateTime = SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(Date())
                    val newName = "\${packageName}_\${gitBranch}_\${type}_\${versionName}_\${dateTime}_\${versionCode}.apk"
                    val newFile = File(modifiedDir, newName)
                    apkFile.copyTo(newFile, overwrite = true)
                    println("Copied \${apkFile.name} to \${newFile.absolutePath}")
                }
            } else {
                println("APK directory for build type '\$type' does not exist: \${apkDir.absolutePath}")
            }
        }
    }
}

tasks.matching { it.name.startsWith("assemble") }.configureEach {
    finalizedBy("moveAndRenameApk")
}
```

### 2️⃣ Required Imports  
Make sure your `build.gradle.kts` has these imports at the top:  

```kotlin
import java.io.File
import java.io.ByteArrayOutputStream
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale
```

---

## 🚀 How to Use  

1️⃣ **Run your build command:**  
```sh
./gradlew assembleDebug    # or assembleRelease
```

2️⃣ **Check the output:**  
- The original APK stays in **`app/build/outputs/apk/debug/`**  
- The renamed APK moves to **`app/build/outputs/apk/debug/modified/`**  

---

## 📂 Example Output  

If your package name is `com.example.app`, you’re on the `main` branch, and your version is `1.0`, the renamed APK will look like this:  

```
app/build/outputs/apk/debug/modified/
└── com_example_app_main_debug_1.0_20250301_123456_1.apk
```

Where:  
- `com_example_app` → Package name  
- `main` → Git branch  
- `debug` → Build type  
- `1.0` → Version name  
- `20250301_123456` → Date and time (YYYYMMDD_HHMMSS)  
- `1` → Version code  

---

## 🔧 Troubleshooting  

❌ **Git error: `Failed to retrieve Git branch`**  
- Make sure Git is installed and your project is inside a valid Git repo.  
- Run `git status` in your terminal to check.  

❌ **APK not renamed?**  
- Ensure you’re running the correct build variant (`debug` or `release`).  
- Check if the APK exists in `app/build/outputs/apk/{debug|release}/`.  

---

## 💡 Future Improvements  

🛠️ Support for product flavors  
🛠️ Option to customize naming via Gradle properties  
🛠️ Improved error handling  

---

## 🤝 Contributions  

**New features, suggestions, and improvements are welcome!**  
- Open an **issue** if you find a bug  
- Submit a **pull request** if you have a fix or enhancement  


---

### 🌟 If this helps, don’t forget to star ⭐ the repo!  
