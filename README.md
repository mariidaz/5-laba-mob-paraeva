# Side A — Сторона А (Native Android · Java)

Мобільний щоденник музичних відкриттів, реалізований з нуля засобами нативної розробки під Android на мові **Java** (Android SDK, Fragments, ViewModel, LiveData, SharedPreferences, HttpURLConnection). Застосунок допомагає шукати альбоми, зберігати їх у власній бібліотеці та записувати оцінки, настрій і особисті враження після прослуховування.

## Запуск

### Спосіб 1. Прямий запуск на Java 17 із терміналу (без Android Studio)

Потрібен встановлений **JDK 17+**. У корені проєкту додано нативний Java-раннер `SideADesktopApp.java`, який відкриває мобільне вікно застосунку напряму через JVM:

```powershell
javac -encoding UTF-8 -d build SideADesktopApp.java
java -cp build SideADesktopApp
```

### Спосіб 2. Через Android Studio (Android Emulator)

Потрібна **Android Studio** з встановленим **Android SDK 34** та **JDK 17**. API key чи фізичний смартфон Android не потрібні — застосунок запускається на вбудованому в Android Studio віртуальному пристрої (**Android Emulator / AVD**).

1. Відкрий теку проєкту `5` у **Android Studio** (`File -> Open`).
2. Дочекайся завершення **Gradle Sync**.
3. Обери віртуальний пристрій у **Device Manager** (Android 8.0 / API 26+) і натисни **Run (`Shift + F10`)**.

Або через термінал:

```powershell
./gradlew assembleDebug
./gradlew testDebugUnitTest
```

## Функції

- **Пошук альбомів і виконавців** у каталозі Apple Music через відкритий **iTunes Search API** (`country=ua`, із автоматичною фільтрацією російських виконавців та специфічних літер рос. алфавіту); API key не потрібен.
- **Каталог** показує до 30 результатів одного запиту та підтримує швидкі жанрові фільтри (`українське`, `інді`, `shoegaze`, `dream pop`, `альтернатива`).
- **Картка альбому** з обкладинкою високої роздільної здатності (`600x600bb`), жанром, роком виходу, кількістю треків і прямим переходом на сторінку альбому в Apple Music через нативний `Intent(Intent.ACTION_VIEW)`.
- **Екран виконавця** з автоматичною підбіркою його альбомів із каталогу.
- **Бібліотека збережених альбомів** та **щоденник прослуховування**: текстова нотатка (до 800 символів), оцінка від 1 до 5 зірок і вибір настрою платівки (`Тепло`, `Ностальгія`, `Спокій`, `Енергія`).
- **Локальне збереження даних** на пристрої через нативний `SharedPreferences` (`side-a-library-v1`) із JSON-серіалізацією (`org.json`).
- **Повна обробка станів мережі**: індикатор завантаження, стан помилки з кнопкою повторної спроби та стан порожнього результату.

## Архітектура та структура проєкту (MVVM · Java)

- `app/src/main/AndroidManifest.xml` — конфігурація застосунку, дозвіл `INTERNET` та реєстрація `MainActivity`.
- `app/src/main/java/ua/sidea/app/MainActivity.java` — точка входу (`AppCompatActivity`), керування нижньою навігацією (`BottomNavigationView`: «Відкривати», «Щоденник», «Про проєкт») та стеком фрагментів (`FragmentManager`).
- `app/src/main/java/ua/sidea/app/data/model/`
  - `Album.java` — POJO-модель альбому з обчисленням року релізу та JSON-серіалізацією.
  - `AlbumNote.java` — POJO-модель нотатки прослуховування з форматуванням дати (`uk-UA`) та JSON-серіалізацією.
- `app/src/main/java/ua/sidea/app/data/remote/ItunesService.java` — багатопотоковий HTTP-клієнт на чистій Java (`ExecutorService`, `HttpURLConnection`, `Handler(Looper.getMainLooper())`) до iTunes Search API з фільтрацією російського контенту.
- `app/src/main/java/ua/sidea/app/data/local/LibraryStorage.java` — шар локального збереження бібліотеки та нотаток у `SharedPreferences`.
- `app/src/main/java/ua/sidea/app/viewmodel/`
  - `DiscoverViewModel.java` — керування станом пошуку каталогу (`LiveData<DiscoverUiState>`).
  - `LibraryViewModel.java` — керування збереженими альбомами та нотатками (`LiveData<LibraryUiState>`) з автоматичною синхронізацією у `LibraryStorage`.
- `app/src/main/java/ua/sidea/app/ui/theme/AppTheme.java` — рожево-біла кольорова палітра, конвертація `dp`/`sp`, генератори `GradientDrawable` та типографіка (`Serif`, `Sans-Serif`, `Monospace`).
- `app/src/main/java/ua/sidea/app/ui/components/`
  - `ImageLoader.java` — асинхронне завантаження обкладинок з кешем у пам’яті (`LruCache<String, Bitmap>`).
  - `UiComponents.java` — перевикористовувані компоненти інтерфейсу (`createAlbumArtwork` із вініловим фолбеком, `createAlbumRow`, `createPageHeader`, `createSectionHeading`).
- `app/src/main/java/ua/sidea/app/ui/screens/` — екрани застосунку на базі `Fragment`:
  - `DiscoverFragment.java`, `JournalFragment.java`, `AboutFragment.java`, `AlbumDetailFragment.java`, `ArtistFragment.java`, `NoteEditorFragment.java`.
- `app/src/test/java/ua/sidea/app/ItunesFilterTest.java` — модульні тести JUnit 4 на мові Java.

## Технології

Java 17, Android SDK (API 26–34), AndroidX AppCompat & Fragment, Material Components (`BottomNavigationView`), AndroidX Lifecycle (`ViewModel` & `LiveData`), `SharedPreferences`, `HttpURLConnection`, `ExecutorService`, `LruCache`, `org.json`, JUnit 4.

## Дизайн

Палітра: чистий білий `#FFFFFF`, ніжно-рожевий `#FFF5F9`, глибокий темно-сливовий `#3B122B` та яскравий рожевий акцент `#EC4899` із пастельно-рожевими лініями `#F9D8E8`. Інтерфейс виконаний у рожево-білому журнальному стилі з акцентною типографікою.
