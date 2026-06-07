name: Android CI (Build From WeTransfer)

on:
  workflow_dispatch: 

# هنا بنحل مشكلة التحذير ونفعل Node 24 أوتوماتيك
env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Install dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y curl unzip

    - name: Download Project ZIP from WeTransfer
      run: |
        curl -L -o project.zip "https://we.tl/t-UrMYLcjmcAPEg85G"

    - name: Unzip Project Files
      run: |
        unzip project.zip -d temp_project
        if [ -d "temp_project/advertising (5)" ]; then
          mv temp_project/advertising\ \(5\)/* .
        elif [ -d "temp_project/app" ]; then
          mv temp_project/* .
        else
          mv temp_project/* .
        fi
        # تنظيف الفولدرات القديمة والثقيلة فوراً لتوفير مساحة الذاكرة والـ RAM
        rm -rf temp_project project.zip .gradle .build-outputs build app/build

    - name: set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Grant execute permission for gradlew
      run: |
        if [ -f "gradlew" ]; then
          chmod +x gradlew
        else
          echo "gradlew not found, generating one..."
          gradle wrapper
          chmod +x gradlew
        fi

    # تحجيم استهلاك الذاكرة (RAM) عشان السيرفر ما يفصلش ويموت
    - name: Build with Gradle
      run: ./gradlew assembleDebug -Dorg.gradle.jvmargs="-Xmx2048m -XX:MaxMetaspaceSize=512m" --no-daemon

    - name: Upload APK
      uses: actions/upload-artifact@v4
      with:
        name: yafta-app-debug-apk
        path: app/build/outputs/apk/debug/app-debug.apk
