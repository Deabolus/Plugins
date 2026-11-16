# Plugins
ОБЩЕЕ. Плагин — это JSON (или просто HTML). Поля: type, id, name, description, version (число), updateUrl (ссылка на свежую версию), permissions.

СКРИПТ (type: "script", поле code с JavaScript). Работает в фоне, API — объект Launcher:
Launcher.toast(текст)
Launcher.island(заголовок, текст, эмодзи, мс, метка)   [island] — если задана метка, по нажатию на остров придёт событие islandTap(метка)
Launcher.islandPin(ключ, заголовок, текст, эмодзи) / Launcher.islandUnpin(ключ)   [island] — постоянная надпись на острове (таймер, счётчик)
Launcher.apps() → [{label,pkg}]   [apps]
Launcher.launch(пакет)   [apps]
Launcher.openUrl(url)
Launcher.get(ключ, по_умолчанию) / Launcher.set(ключ, значение) — своё хранилище
Launcher.setting(ключ) / Launcher.setSetting(ключ, значение)   [settings] (ключ как в пресетах; для обоев — "wallpaper.mode")
Launcher.http(url) → текст   [net]
Launcher.battery() → {level, charging}
Launcher.vibrate(мс)
Launcher.addMenu(название, функция) — пункт в меню долгого нажатия на рабочем столе
Launcher.prompt(заголовок, функция(текст)) / Launcher.choose(заголовок, [варианты], функция(номер))
Launcher.every(секунды, функция)
Launcher.on("start"|"resume"|"appLaunch"|"notification"|"media"|"charging"|"batteryLow"|"islandTap", функция(данные))
media → {title, artist, playing, pkg}; charging и batteryLow → {level}
Пример: {"type":"script","name":"Привет","permissions":["island"],"code":"Launcher.addMenu('Привет',function(){Launcher.island('Привет','Работает','👋',3000)})"}

ТЕМА ОСТРОВА (type "island") — HTML-страница, которая рисует развёрнутый остров, без пересборки лаунчера. Проще всего обычный HTML-файл с тегами <meta name="launcher-type" content="island">, <meta name="island-aspect" content="1.5"> (ширина к высоте) и <title>. Или JSON: {"type":"island","name":"…","aspect":1.5,"width":340,"permissions":["net"],"html":"…"}. Выбор: Настройки → Остров → «Выбрать тему-плагин».
API темы (объект Island): Island.media() → JSON-строка {title, artist, album, playing, pos, dur (мс), speed, t (время замера, мс), pkg, hasArt}; текущая позиция = pos + (Date.now()-t)*speed. Island.art() → обложка в виде data-ссылки (или пусто). Island.accent() → "#rrggbb". Island.control("play"|"pause"|"toggle"|"next"|"prev"|"open"|"collapse"). Island.seek(0..1). Island.log(текст). Если задать window.__island = function(){...}, лаунчер вызывает её каждые 0,5 с. Интернет в теме выключен, пока нет разрешения "net". Образцы: «Остров-тема: Lo-fi комната» и «минимал» (Плагины → «Установить примеры»).

БЫСТРЫЕ ДЕЙСТВИЯ: Launcher.quick(id) [system] и Launcher.quickList() → [{id,title,emoji,state}]; в теме острова — Island.action(id) и Island.actions(). id: flash, wifi, data, bt, airplane, dnd, ringer, rotate, bright, vol_up, vol_down, mute, lock, shot, qs, power, nfc, loc, saver. Wi‑Fi, Bluetooth, полёт и интернет Android не даёт переключать напрямую: открывается системная панель.

НАТИВНЫЙ ДОСТУП («установщик розеток»): скрипту с разрешением "native" открыт Launcher.android — прямые вызовы Android и лаунчера. Включить: Настройки → Прочее → «Разрешить плагинам нативный доступ». A = Launcher.android; act = A.activity() — главный экран; A.slot("drawerTop"|"homeOverlay"|"canvas"|"root"|"dock") — готовые розетки (контейнеры для ваших View); A.create("android.widget.Button", act) — создать объект; obj.call("метод", арг…), obj.get("поле"), obj.set("поле", значение), obj.release(); A.call("android.graphics.Color", "parseColor", "#ff00ff") — статический вызов; A.field("android.view.Gravity", "CENTER") — статическое поле. Функции в аргументах становятся слушателями (View.OnClickListener и т.п.); {fn: функция, ret: true} — если слушатель должен вернуть true. Launcher.onInstall(функция) выполняется один раз при установке или обновлении плагина, Launcher.onUninstall(функция) — при удалении. Содержимое розеток пересоздаётся при каждом запуске, поэтому добавляйте его в основном коде скрипта. Образец: «Установщик: кнопка в шапке списка приложений». ВНИМАНИЕ: такой плагин может всё, что может сам лаунчер.

НАСТРОЙКИ ПЛАГИНА: в JSON плагина поле "settings":[{"key":"speed","type":"slider","title":"Скорость","min":1,"max":10,"default":5,"unit":" x"},{"key":"on","type":"toggle","title":"Включено","default":true},{"key":"mode","type":"choice","title":"Режим","options":["А","Б"],"default":0},{"key":"clr","type":"color","title":"Цвет","default":"#a58fe6"},{"key":"txt","type":"text","title":"Подпись","default":"привет"}]. У одиночного HTML-файла: <script type="application/json" id="launcher-settings">[…]</script>. Настройки появляются в Настройки → «Настройки плагинов» и в меню плагина. Чтение: скрипты — Launcher.pref(ключ), Launcher.setPref(ключ, значение), событие Launcher.on("pref", функция({id,key,value})); темы острова — функции pref(ключ, по_умолчанию) и prefs(), а при изменении вызывается window.__pref = function(ключ, значение){...}.
СЛОТЫ ТЕМЫ ОСТРОВА: <meta name="island-slot" content="expanded"> — весь развёрнутый остров (по умолчанию). content="controls" и <meta name="island-height" content="56"> — только ряд кнопок плеера в стандартном стиле (Настройки → Остров → «Тема кнопок плеера (плагин)»).
Launcher.addAppMenu(название, функция(пакет)) — пункт в меню долгого нажатия на приложение.

ОСТРОВ (ключи для Launcher.setSetting и пресетов, числа в dp или %): islandW, islandH (капсула), islandExpW (развёрнутая), islandEventW (уведомление), islandX, islandY (смещение), islandRadius (%), islandAlpha (%), islandTextScale (%), islandBgMode (0 чёрный, 1 серый, 2 акцент), islandAnimMs, islandTap, islandLong (0..3, действие), islandEventMs (сек); флаги: islandShowArt, islandShowEq, islandShowProgress, islandStroke, islandVibrate, islandNotifText.

HTML-ВИДЖЕТ: {"type":"html","name":"Имя","w":220,"h":220,"html":"<div>...</div>"} — доступен Launcher.battery(), openUrl, openApp, toast. Виджет на весь экран работает как анимированный фон рабочего стола.

КНОПКИ: {"type":"buttons","columns":3,"buttons":[{"label":"YouTube","icon":"▶️","url":"https://youtube.com"},{"label":"Telegram","app":"org.telegram.messenger"}]}

ПРЕСЕТ: {"type":"preset","launcher":{"accent":-16720385,"dim":20},"wallpaper":{"mode":"aurora","speed":6}}

РЕПОЗИТОРИЙ: index.json вида {"plugins":["a.json","b.json"]} (ссылки можно писать относительно index.json). Укажите ссылку на него в разделе «Обновления» — плагины будут ставиться и обновляться сами, если поднять version.
