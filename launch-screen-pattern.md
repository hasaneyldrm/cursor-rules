# 📱 React Native: LaunchScreen Extended Pattern

## 🎯 Amaç

Uygulama açılırken kullanıcıya daha iyi bir deneyim sunmak için:
- ❌ Ekstra loading screen göstermemek
- ✅ Native LaunchScreen'i data yüklenene kadar göstermek
- ✅ Smooth ve profesyonel geçiş yapmak
- ✅ Uygulama açılırken kritik verileri önceden yüklemek

---

## 📦 1. Gerekli Paket

```bash
npm install react-native-splash-screen
# veya
yarn add react-native-splash-screen
```

### iOS için Pod Install
```bash
cd ios && pod install && cd ..
```

---

## 🔧 2. iOS Native Entegrasyonu

### 2.1. AppDelegate Düzenleme

**Objective-C kullanıyorsan:** `ios/YourApp/AppDelegate.mm`

```objc
#import "AppDelegate.h"
#import <React/RCTBundleURLProvider.h>
#import "RNSplashScreen.h"  // ✅ Import ekle

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application 
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  // ... React Native setup ...
  
  [RNSplashScreen show];  // ✅ SplashScreen'i göster
  
  return YES;
}

@end
```

---

**Swift kullanıyorsan:** `ios/YourApp/AppDelegate.swift`

```swift
import UIKit
import RNSplashScreen  // ✅ Import ekle

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  
  var window: UIWindow?
  
  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    
    // React Native bridge setup...
    let bridge = RCTBridge(delegate: self, launchOptions: launchOptions)
    let rootView = RCTRootView(bridge: bridge!, moduleName: "YourApp", initialProperties: nil)
    
    let rootViewController = UIViewController()
    rootViewController.view = rootView
    
    self.window = UIWindow(frame: UIScreen.main.bounds)
    self.window?.rootViewController = rootViewController
    self.window?.makeKeyAndVisible()
    
    // ✅ SplashScreen'i göster (LaunchScreen'i uzat)
    RNSplashScreen.show()
    
    return true
  }
}
```

---

### 2.2. Bridging Header (Sadece Swift için)

Eğer Swift kullanıyorsan ve Bridging Header yoksa:

1. Xcode'da `File` → `New` → `File` → `Header File`
2. İsim: `YourApp-Bridging-Header.h`

**İçeriği:**
```objc
#import <React/RCTBridgeModule.h>
#import <React/RCTBridge.h>
#import <React/RCTEventDispatcher.h>
#import <React/RCTRootView.h>
#import <React/RCTUtils.h>
#import <React/RCTConvert.h>
#import <React/RCTBundleURLProvider.h>
#import "RNSplashScreen.h"  // ✅ SplashScreen import
```

3. **Build Settings**'de ara: `Objective-C Bridging Header`
4. Value: `YourApp/YourApp-Bridging-Header.h`

---

## 🤖 3. Android Native Entegrasyonu

### 3.1. MainActivity Düzenleme

`android/app/src/main/java/com/yourapp/MainActivity.java`:

```java
package com.yourapp;

import android.os.Bundle;
import com.facebook.react.ReactActivity;
import com.facebook.react.ReactActivityDelegate;
import com.facebook.react.defaults.DefaultNewArchitectureEntryPoint;
import com.facebook.react.defaults.DefaultReactActivityDelegate;
import org.devio.rn.splashscreen.SplashScreen; // ✅ Import ekle

public class MainActivity extends ReactActivity {

  @Override
  protected String getMainComponentName() {
    return "YourApp";
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    SplashScreen.show(this);  // ✅ LaunchScreen'i göster
    super.onCreate(savedInstanceState);
  }

  @Override
  protected ReactActivityDelegate createReactActivityDelegate() {
    return new DefaultReactActivityDelegate(
      this,
      getMainComponentName(),
      DefaultNewArchitectureEntryPoint.getFabricEnabled()
    );
  }
}
```

---

### 3.2. Launch Screen XML (Android)

`android/app/src/main/res/layout/launch_screen.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:gravity="center"
    android:orientation="vertical">

    <ImageView
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:src="@drawable/launch_screen" />

</LinearLayout>
```

Logo'yu `android/app/src/main/res/drawable/` klasörüne ekle.

---

## ⚛️ 4. React Native Tarafı

### 4.1. Ana App Component

**src/App.tsx:**

```typescript
import React, { useEffect, useState } from 'react';
import { View, ActivityIndicator } from 'react-native';
import SplashScreen from 'react-native-splash-screen';
import { NavigationContainer } from '@react-navigation/native';
import { RootNavigator } from './navigation/RootNavigator';

const App = () => {
  const [isAppReady, setIsAppReady] = useState(false);

  useEffect(() => {
    initializeApp();
  }, []);

  const initializeApp = async () => {
    try {
      // ✅ 1. Kritik verileri paralel olarak yükle
      await Promise.all([
        fetchUserInfo(),      // Kullanıcı bilgisi
        fetchAppConfig(),     // App konfigürasyonu
        loadCachedData(),     // Cache'den data
        initializeServices(), // Push notification, analytics vs
      ]);

      console.log('✅ App initialization completed');
      
      // ✅ 2. Data hazır, LaunchScreen'i kapat
      SplashScreen.hide();
      
      // ✅ 3. App render'a hazır
      setIsAppReady(true);
      
    } catch (error) {
      console.error('❌ App initialization error:', error);
      
      // ⚠️ Hata olsa bile splash'ı kapat (sonsuz bekletme olmasın)
      SplashScreen.hide();
      setIsAppReady(true);
    }
  };

  // Fetch fonksiyonları
  const fetchUserInfo = async () => {
    // API call
    await new Promise(resolve => setTimeout(resolve, 1000));
  };

  const fetchAppConfig = async () => {
    // API call
    await new Promise(resolve => setTimeout(resolve, 500));
  };

  const loadCachedData = async () => {
    // AsyncStorage'dan data yükle
    await new Promise(resolve => setTimeout(resolve, 300));
  };

  const initializeServices = async () => {
    // OneSignal, Firebase, Analytics vs.
    await new Promise(resolve => setTimeout(resolve, 500));
  };

  // ✅ Data yüklenene kadar null render et
  // LaunchScreen bu süre boyunca gözükmeye devam eder
  if (!isAppReady) {
    return null;
  }

  return (
    <NavigationContainer>
      <RootNavigator />
    </NavigationContainer>
  );
};

export default App;
```

---

### 4.2. Context/Provider Pattern (Opsiyonel)

Eğer app-wide state kullanıyorsan:

**src/App.tsx:**

```typescript
import React, { useEffect, useState } from 'react';
import SplashScreen from 'react-native-splash-screen';
import { NavigationContainer } from '@react-navigation/native';
import { AppProvider } from './context/AppContext';
import { RootNavigator } from './navigation/RootNavigator';

const App = () => {
  const [isAppReady, setIsAppReady] = useState(false);

  useEffect(() => {
    initializeApp();
  }, []);

  const initializeApp = async () => {
    try {
      await Promise.all([
        fetchUserInfo(),
        fetchAppConfig(),
      ]);

      SplashScreen.hide();
      setIsAppReady(true);
      
    } catch (error) {
      console.error('App init error:', error);
      SplashScreen.hide();
      setIsAppReady(true);
    }
  };

  const fetchUserInfo = async () => {
    // API call
  };

  const fetchAppConfig = async () => {
    // API call
  };

  if (!isAppReady) {
    return null;
  }

  return (
    <AppProvider>
      <NavigationContainer>
        <RootNavigator />
      </NavigationContainer>
    </AppProvider>
  );
};

export default App;
```

---

## 🎨 5. LaunchScreen Tasarımı

### 5.1. iOS: LaunchScreen.storyboard

1. Xcode'da `ios/YourApp.xcworkspace` aç
2. `LaunchScreen.storyboard` dosyasını seç
3. Interface Builder'da:
   - **ImageView** ekle (Logo için)
   - **Constraints** ayarla:
     - Center X = Superview Center X
     - Center Y = Superview Center Y
     - Width = 200, Height = 200 (sabit boyut)
   - **Background Color** ayarla
4. Logo görseli:
   - `Assets.xcassets` içine `LaunchScreen` image set ekle
   - 1x, 2x, 3x versiyonlarını ekle

**⚠️ Önemli Kısıtlamalar:**
- ❌ Custom font kullanılamaz
- ❌ Animation yapılamaz
- ❌ Network request olmaz
- ❌ Dinamik content olmaz
- ✅ Sadece statik görsel ve renkler

---

### 5.2. Android: Splash Screen Theme

`android/app/src/main/res/values/styles.xml`:

```xml
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.DayNight.NoActionBar">
        <item name="android:editTextBackground">@drawable/rn_edit_text_material</item>
    </style>

    <!-- ✅ Splash Screen Theme -->
    <style name="SplashTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowBackground">@drawable/launch_screen</item>
    </style>
</resources>
```

`android/app/src/main/AndroidManifest.xml`:

```xml
<application
    android:name=".MainApplication"
    android:theme="@style/AppTheme">
    
    <!-- ✅ MainActivity'ye SplashTheme uygula -->
    <activity
        android:name=".MainActivity"
        android:theme="@style/SplashTheme"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

---

## 📊 6. Akış Diyagramı

```
┌─────────────────────────────────────────┐
│  App Açılışı (Native)                   │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  LaunchScreen Gösteriliyor              │
│  (Native Storyboard/XML)                │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  AppDelegate: RNSplashScreen.show()     │
│  (LaunchScreen uzatılıyor)              │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  React Native Bundle Yükleniyor         │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  App.tsx: initializeApp() Başlıyor     │
│  • fetchUserInfo()                      │
│  • fetchAppConfig()                     │
│  • loadCachedData()                     │
│  • initializeServices()                 │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  Tüm Data Hazır                         │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  SplashScreen.hide()                    │
│  setIsAppReady(true)                    │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  NavigationContainer Render             │
│  Ana Ekran Gösteriliyor                 │
└─────────────────────────────────────────┘
```

---

## ⚡ 7. Best Practices

### 7.1. Timeout Ekle
```typescript
const initializeApp = async () => {
  try {
    // ✅ Maximum 5 saniye bekle
    await Promise.race([
      Promise.all([
        fetchUserInfo(),
        fetchAppConfig(),
      ]),
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Timeout')), 5000)
      )
    ]);

    SplashScreen.hide();
    setIsAppReady(true);
    
  } catch (error) {
    console.error('App init error:', error);
    SplashScreen.hide();
    setIsAppReady(true);
  }
};
```

### 7.2. Error Handling
```typescript
const fetchUserInfo = async () => {
  try {
    const response = await api.get('/user/info');
    await AsyncStorage.setItem('userInfo', JSON.stringify(response.data));
    return response.data;
  } catch (error) {
    // ⚠️ Hata olsa bile app açılsın
    console.warn('User info fetch failed:', error);
    return null;
  }
};
```

### 7.3. Caching Strategy
```typescript
const loadCachedData = async () => {
  try {
    // ✅ Önce cache'den yükle
    const cached = await AsyncStorage.getItem('userInfo');
    if (cached) {
      // Cache'li data'yı kullan
      return JSON.parse(cached);
    }
  } catch (error) {
    console.warn('Cache load failed:', error);
  }
  return null;
};
```

---

## 🐛 8. Troubleshooting

### iOS: "RNSplashScreen.h file not found"
```bash
cd ios
rm -rf Pods Podfile.lock
pod install
cd ..
npx react-native run-ios
```

### iOS: Swift Bridging Header Hatası
- Build Settings → Objective-C Bridging Header
- Path: `YourApp/YourApp-Bridging-Header.h`
- Clean Build: Cmd+Shift+K

### Android: "SplashScreen cannot be resolved"
`android/app/build.gradle`:
```gradle
dependencies {
    implementation project(':react-native-splash-screen')
}
```

`android/settings.gradle`:
```gradle
include ':react-native-splash-screen'
project(':react-native-splash-screen').projectDir = 
    new File(rootProject.projectDir, '../node_modules/react-native-splash-screen/android')
```

### SplashScreen Kapanmıyor
```typescript
// ✅ Her durumda kapatıldığından emin ol
useEffect(() => {
  const timer = setTimeout(() => {
    SplashScreen.hide();
  }, 10000); // 10 saniye max

  return () => clearTimeout(timer);
}, []);
```

---

## 📝 9. Checklist

### iOS
- [ ] `react-native-splash-screen` yüklendi
- [ ] `pod install` yapıldı
- [ ] `AppDelegate` düzenlendi (`RNSplashScreen.show()`)
- [ ] Bridging Header oluşturuldu (Swift ise)
- [ ] `LaunchScreen.storyboard` düzenlendi
- [ ] Logo eklendi
- [ ] Build ve test edildi

### Android
- [ ] `react-native-splash-screen` yüklendi
- [ ] `MainActivity` düzenlendi
- [ ] `launch_screen.xml` oluşturuldu
- [ ] Logo `drawable/` klasörüne eklendi
- [ ] `styles.xml` düzenlendi
- [ ] `AndroidManifest.xml` düzenlendi
- [ ] Build ve test edildi

### React Native
- [ ] `App.tsx` düzenlendi
- [ ] `initializeApp()` fonksiyonu oluşturuldu
- [ ] Kritik API'ler belirlendi
- [ ] `SplashScreen.hide()` eklendi
- [ ] Error handling eklendi
- [ ] Timeout mekanizması eklendi
- [ ] Test edildi

---

## 🚀 10. Sonuç

Bu pattern ile:
- ✅ Kullanıcı loading screen görmez
- ✅ Native LaunchScreen smooth bir şekilde uzatılır
- ✅ App açılırken data hazır olur
- ✅ Profesyonel bir UX sağlanır

**Örnek Kullanım:**
- Instagram, WhatsApp, Twitter gibi popüler uygulamalar bu pattern'i kullanır
- Kullanıcı hiç loading görmeden direkt ana ekrana geçer
- Tüm kritik data arka planda yüklenmiş olur

---

## 📚 Kaynaklar

- [react-native-splash-screen](https://github.com/crazycodeboy/react-native-splash-screen)
- [iOS Human Interface Guidelines - Launch Screen](https://developer.apple.com/design/human-interface-guidelines/launching)
- [Android - Splash Screens](https://developer.android.com/develop/ui/views/launch/splash-screen)

---

**Son Güncelleme:** Kasım 2025
**React Native Version:** 0.80+
**Pattern:** LaunchScreen Extended


