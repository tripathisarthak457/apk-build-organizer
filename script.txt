import java.io.ByteArrayOutputStream
import java.io.File
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

tasks.register("moveAndRenameApk") {
    group = "custom"
    description = "Deletes old files in the 'modified' folder and copies the generated APKs with new names, including Git branch info and date/time."

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
            val apkDir = file("${buildDirFile}/outputs/apk/$type")
            if (apkDir.exists()) {
                val modifiedDir = File(apkDir, "modified")
                if (modifiedDir.exists()) {
                    modifiedDir.listFiles()?.forEach { file ->
                        if (file.isFile) {
                            file.delete()
                        }
                    }
                } else {
                    modifiedDir.mkdirs()
                }
                apkDir.listFiles { file -> file.name.endsWith(".apk") }?.forEach { apkFile ->

                    val packageName = android.defaultConfig.applicationId?.replace(".", "_")
                    val versionName = android.defaultConfig.versionName
                    val versionCode = android.defaultConfig.versionCode
                    val dateTime = SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(Date())
                    val newName = "${packageName}_${gitBranch}_${type}_${versionName}_${dateTime}_${versionCode}.apk"
                    val newFile = File(modifiedDir, newName)
                    apkFile.copyTo(newFile, overwrite = true)
                    println("Copied ${apkFile.name} to ${newFile.absolutePath}")
                }
            } else {
                println("APK directory for build type '$type' does not exist: ${apkDir.absolutePath}")
            }
        }
    }
}

tasks.matching { it.name.startsWith("assemble") }.configureEach {
    finalizedBy("moveAndRenameApk")
}
