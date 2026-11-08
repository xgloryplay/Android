set -e
cd "$(git rev-parse --show-toplevel)"

npm init -y >/dev/null
npm i -D @capacitor/cli@6.2.3 >/dev/null
npm i @capacitor/core@6.2.3 @capacitor/android@6.2.3 >/dev/null

cat > capacitor.config.json <<'EOF'
{
  "appId": "com.satuproyeksi.reconciler",
  "appName": "Reconciler",
  "webDir": "www",
  "server": {
    "url": "https://andalanpratamaindonesia.com/recon/pilihan.php",
    "androidScheme": "https",
    "cleartext": false
  }
}
EOF
mkdir -p www && echo '<!doctype html><meta charset="utf-8"><title>Reconciler</title><p>Memuat...</p>' > www/index.html

npx cap add android

mkdir -p android/app/src/main/res/xml
cat > android/app/src/main/AndroidManifest.xml <<'EOF'
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <uses-feature android:name="android.hardware.camera" android:required="false" />
    <uses-feature android:name="android.hardware.camera.front" android:required="false" />
    <uses-feature android:name="android.hardware.location.gps" android:required="false" />

    <application
        android:allowBackup="false"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="Reconciler"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:networkSecurityConfig="@xml/network_security_config"
        android:usesCleartextTraffic="false">

        <activity
            android:name=".MainActivity"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|smallestScreenSize|screenLayout|uiMode|navigation"
            android:exported="true"
            android:launchMode="singleTask"
            android:label="Reconciler"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
EOF
cat > android/app/src/main/res/xml/network_security_config.xml <<'EOF'
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">andalanpratamaindonesia.com</domain>
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </domain-config>
</network-security-config>
EOF

mkdir -p .github/workflows
cat > .github/workflows/build-apk.yml <<'EOF'
name: Build APK

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Install dependencies
        run: npm install

      - name: Sync Capacitor
        run: npx cap sync android

      - name: Build debug APK
        working-directory: android
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug

      - uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: android/app/build/outputs/apk/debug/app-debug.apk
EOF

cat >> .gitignore <<'EOF'
node_modules/
android/app/build/
android/.gradle/
android/local.properties
EOF

git add -A
git commit -m "Setup APK Reconciler: Capacitor, izin kamera & GPS, workflow build"
git push origin HEAD
echo "Selesai. Pantau build di tab Actions."
