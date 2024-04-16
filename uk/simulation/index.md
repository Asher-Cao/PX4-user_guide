# Симулятор

Симулятори дозволяють польотному коду PX4 керувати комп'ютерно змодельованим апаратом у змодельованому "світі". Ви можете взаємодіяти з цим апаратом так само, як і зі справжнім, використовуючи _QGroundControl_, позабортовий API або радіоконтролер/ігровий пульт.

:::tip
Симуляція - це швидкий, простий, а головне, _безпечний_ спосіб протестувати зміни в коді PX4 перед тим, як спробувати літати в реальному світі. Це також хороший спосіб почати літати з PX4, якщо у вас ще немає апарату для експериментів.
:::

PX4 підтримує як симуляцію _Software In the Loop (SITL)_, де польотний стек працює на комп'ютері (або на тому ж комп'ютері, або на іншому комп'ютері в тій же мережі), так і симуляцію _Hardware In the Loop (HITL)_ з використанням симуляційної прошивки на реальній платі польотного контролера.

Інформація про доступні тренажери та способи їх налаштування наведена в наступному розділі. Інші розділи надають загальну інформацію про те, як працює симулятор, і не є обов'язковими для _використання_ симуляторів.

## Підтримувані симулятори

Наступні симулятори підтримуються основною командою розробників PX4.


| Симулятор                                        | Опис                                                                                                                                           |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [Gazebo](../sim_gazebo_gz/index.md)              | <p><strong>Ми дуже рекомендуємо цей тренажер.</strong></p><p>Gazebo supersedes [Gazebo Classic](../sim_gazebo_classic/README.md), з більш досконалими моделями рендерингу, фізики та сенсорів. Це єдина версія Gazebo, доступна з Ubuntu Linux 22.04.</p><p>Потужне середовище 3D-симуляції, яке особливо підходить для тестування обходу перешкод та комп'ютерного зору. Він також може бути використаний для [multi-vehicle simulation](../simulation/multi-vehicle-simulation.md) і зазвичай використовується з [ROS](../simulation/ros_interface.md), набором інструментів для автоматизації керування апаратами. </p><p><strong> Підтримувані апарати:</strong> Quad, Standard VTOL, Літак</p>                                           |
| [Gazebo Classic](../sim_gazebo_classic/index.md) | <p><strong>Ми дуже рекомендуємо цей тренажер.</strong></p><p>Потужне середовище 3D-симуляції, яке особливо підходить для тестування обходу перешкод та комп'ютерного зору. Він також може бути використаний для [multi-vehicle simulation](../simulation/multi-vehicle-simulation.md) і зазвичай використовується з [ROS](../simulation/ros_interface.md), набором інструментів для автоматизації керування апаратами.</p><p><strong>Підтримувані апарати:</strong> Quad ([Iris](../airframes/airframe_reference.md#copter_quadrotor_x_generic_quadcopter), Hex (Typhoon H480), [Generic Standard VTOL (QuadPlane)](../airframes/airframe_reference.md#vtol_standard_vtol_generic_standard_vtol), Tailsitter, Plane, Rover, Submarine </p>                                                                    |
| [jMAVSim](../sim_jmavsim/index.md)               | Простий мультироторний симулятор, який дозволяє літати на _коптерах_ по симульованому світу. <p>Його легко налаштувати і можна використовувати для перевірки того, що ваш апарат може злітати, летіти, приземлятися і належним чином реагувати на різні несправності (наприклад, несправність GPS). Він також може бути використаний для [симуляції декількох апаратів] (../sim_jmavsim/multi_vehicle.md).</p><p><strong> Підтримувані апарати:</strong> Quad</p> |

Існує також ряд [Тренажерів, що підтримуються спільнотою](../simulation/community_supported_simulators.md).

---

Решта цієї теми - це "дещо загальний" опис того, як працює інфраструктура симуляції. Це не обовʼязково для _використання_ симуляторів.

## Симулятор MAVLink API

Всі симулятори, крім Gazebo, взаємодіють з PX4 за допомогою API симулятора MAVLink. Цей API визначає набір повідомлень MAVLink, які передають дані датчиків з модельованого світу в PX4 і повертають значення двигуна і приводу з польотного коду, які будуть застосовані до модельованого апарату. На зображенні нижче показано потік повідомлень.

![Симулятор MAVLink API](../../assets/simulation/px4_simulator_messages.svg)

::: info A SITL build of PX4 uses [SimulatorMavlink.cpp](https://github.com/PX4/PX4-Autopilot/blob/main/src/modules/simulation/simulator_mavlink/SimulatorMavlink.cpp) to handle these messages while a hardware build in HIL mode uses [mavlink_receiver.cpp](https://github.com/PX4/PX4-Autopilot/blob/main/src/modules/mavlink/mavlink_receiver.cpp). Дані датчиків з симулятора записуються в теми PX4 uORB. Всі двигуни/приводи заблоковані, але внутрішнє програмне забезпечення повністю функціонує.
:::

Повідомлення описані нижче (див. посилання для більш детальної інформації).

| Повідомлення                                                        | Напрямок   | Опис                                                                                                                                                                                                                                                                                      |
| ------------------------------------------------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [MAV_MODE:MAV_MODE_FLAG_HIL_ENABLED][mav_mode_flag_hil_enabled] | NA         | Прапорець режиму при використанні симуляції. Всі двигуни/приводи заблоковані, але внутрішнє програмне забезпечення повністю функціонує.                                                                                                                                                   |
| [HIL_ACTUATOR_CONTROLS][hil_actuator_controls]                    | PX4 -> Sim | Виходи керування PX4 (до двигунів, приводів).                                                                                                                                                                                                                                             |
| [HIL_SENSOR][hil_sensor]                                            | Sim -> PX4 | Імітація показань IMU в одиницях СІ в рамі корпусу NED.                                                                                                                                                                                                                                   |
| [HIL_GPS][hil_gps]                                                  | Sim -> PX4 | Симульоване значення датчика GPS RAW.                                                                                                                                                                                                                                                     |
| [HIL_OPTICAL_FLOW][hil_optical_flow]                              | Sim -> PX4 | Імітація оптичного потоку від датчика потоку (наприклад, PX4FLOW або датчика оптичної миші)                                                                                                                                                                                               |
| [HIL_STATE_QUATERNION][hil_state_quaternion]                      | Sim -> PX4 | Містить фактичне "змодельоване" положення апарату, орієнтацію, швидкість і т.д. Це може бути записано в журнал і співставлено з оцінками PX4 для аналізу і діагностики (наприклад, перевірка того, наскільки добре оцінювач працює для зашумлених (імітованих) вхідних сигналів датчика). |
| [HIL_RC_INPUTS_RAW][hil_rc_inputs_raw]                            | Sim -> PX4 | Отримані RAW-значення каналів РК.                                                                                                                                                                                                                                                         |

<!-- links for table above -->



<!-- above ^^^ links for table -->

Безпосереднє використання PX4 [Gazebo API](https://gazebosim.org/docs) для взаємодії з [Gazebo](../sim_gazebo_gz/README.md) і MAVlink не потрібно.

## Порти UDP PX4 MAVLink за замовчуванням

За замовчуванням PX4 використовує загальноприйняті порти UDP для зв'язку MAVLink з наземними станціями управління (наприклад, _QGroundControl_), позабортовими API (наприклад, MAVSDK, MAVROS) і API симуляторів (наприклад, Gazebo). Ці порти:

- Віддалений UDP-порт PX4 **14550** використовується для зв'язку з наземними станціями управління. Очікується, що GCS прослуховуватиме з'єднання на цьому порту. _QGroundControl_ слухає цей порт за замовчуванням.
- Віддалений UDP-порт PX4 **14540** використовується для зв'язку з зовнішніми API. Очікується, що зовнішні API будуть чекати на з'єднання через цей порт. ::: info Multi-vehicle simulations use a separate remote port for each instance, allocated sequentially from `14540` to `14549` (additional instances all use port `14549`).
:::
- Для зв'язку з PX4 використовується локальний TCP-порт симулятора **4560**. Симулятор слухає цей порт, і PX4 ініціює TCP-з'єднання з ним.

::: info The ports for the GCS, offboard APIs and simulator are specified by startup scripts. Щоб дізнатися більше, див. розділ [Запуск системи](../concept/system_startup.md).
:::

<!-- A useful discussion about UDP ports here: https://github.com/PX4/PX4-user_guide/issues/1035#issuecomment-777243106 -->

## Середовище симуляції SITL

На схемі нижче показано типове середовище симуляції SITL для будь-якого з підтримуваних тренажерів, що використовують MAVLink (тобто всіх, окрім Gazebo).

![Огляд PX4 SITL](../../assets/simulation/px4_sitl_overview.svg)

Різні частини системи з'єднуються через протокол UDP і можуть працювати як на одному комп'ютері, так і на іншому комп'ютері в тій самій мережі.

- PX4 використовує спеціальний модуль для підключення до локального TCP-порту 4560 симулятора. Потім симулятори обмінюються інформацією з PX4 за допомогою [Симулятор MAVLink API](#simulator-mavlink-api), описаного вище. PX4 на SITL і симулятор можуть працювати як на одному комп'ютері, так і на різних комп'ютерах в одній мережі.

  ::: info Simulators can also use the _uxrce-dds bridge_ ([XRCE-DDS](../middleware/uxrce_dds.md)) to directly interact with PX4 (i.e. via [UORB topics](../middleware/uorb.md) rather than MAVLink). Цей підхід _може_ використовуватися Gazebo Classic для моделювання [кількох апаратів](../sim_gazebo_classic/multi_vehicle_simulation.md#build-and-test-xrce-dds).
:::

- PX4 використовує звичайний модуль MAVLink для підключення до наземних станцій і зовнішніх API розробників, таких як MAVSDK або ROS
  - Наземні станції слухають віддалений UDP-порт PX4: `14550`
  - Зовнішні API розробника слухають віддалений UDP-порт PX4: `14540`. Для симуляції декількох апаратів PX4 послідовно виділяє окремий віддалений порт для кожного екземпляра від `14540` до `14549` (додаткові екземпляри використовують порт `14549`).
- PX4 визначає ряд _локальних_ UDP-портів (`14580`,`18570`), які іноді використовуються при роботі в мережі з PX4, запущеним у контейнері або віртуальній машині. Вони не рекомендуються для "загального" використання і можуть змінюватися в майбутньому.
- Послідовне з'єднання можна використовувати для підключення [джойстика/геймпада](../config/joystick.md) через _QGroundControl_.

Якщо ви використовуєте звичайну систему побудови SITL `для створення` конфігураційних цілей (див. наступний розділ), то і SITL, і симулятор будуть запущені на одному комп'ютері, і наведені вище порти будуть автоматично налаштовані. Ви можете налаштувати додаткові UDP-з'єднання MAVLink та іншим чином змінити середовище моделювання у файлах конфігурації та ініціалізації збірки.

### Запуск/створення симуляції SITL

Система збірки дозволяє дуже легко зібрати і запустити PX4 на SITL, активувати симулятор і з'єднати їх. Синтаксис (спрощений) виглядає наступним чином:

```sh
make px4_sitl simulator[_vehicle-model]
```

де `simulator` - це `gz` (для Gazebo), `gazebo-classic`, `jmavsim` або інший симулятор, а vehicle-model - це конкретний тип транспортного засобу, який підтримується цим симулятором ([Gazebo](../sim_gazebo_gz/README.md) та [jMAVSim](../sim_jmavsim/README.md) на момент написання статті підтримують лише мультикоптери, тоді як [Gazebo Classic](../sim_gazebo_classic/README.md) підтримує багато різних типів).

Нижче наведено кілька прикладів, і їх набагато більше на окремих сторінках для кожного з симуляторів:

```sh
# Start Gazebo with the x500 multicopter
make px4_sitl gz_x500

# Start Gazebo Classic with plane
make px4_sitl gazebo-classic_plane

# Start Gazebo Classic with iris and optical flow
make px4_sitl gazebo-classic_iris_opt_flow

# Start JMavSim with iris (default vehicle model)
make px4_sitl jmavsim

# Start PX4 with no simulator (i.e. to use your own "custom" simulator)
make px4_sitl none_iris
```

Симуляцію можна додатково налаштувати за допомогою змінних середовища:

- `PX4_ESTIMATOR`: Ця змінна визначає, який естіматор використовувати. Можливі варіанти: `ekf2` (за замовчуванням), `lpe` (застаріла). Його можна встановити через `export PX4_ESTIMATOR=lpe` перед запуском симуляції.

Описаний тут синтаксис є спрощеним, і існує багато інших опцій, які можна налаштувати за допомогою _make_ - наприклад, вказати, що ви бажаєте під'єднатися до IDE або дебаггера. Для отримання додаткової інформації дивіться: [Збірка коду > PX4 Make Build Targets](../dev_setup/building_px4.md#px4-make-build-targets).

<a id="simulation_speed"></a>

### Запуск симуляції швидше, ніж у реальному часі

SITL можна запустити швидше або повільніше, ніж у реальному часі, використовуючи jMAVSim або Gazebo Classic.

Коефіцієнт швидкості задається за допомогою змінної оточення `PX4_SIM_SPEED_FACTOR`. Наприклад, запустити симуляцію jMAVSim зі швидкістю у 2 рази більшою за швидкість реального часу:

```sh
PX4_SIM_SPEED_FACTOR=2 make px4_sitl jmavsim
```

Запустити в половину реального часу:

```sh
PX4_SIM_SPEED_FACTOR=0.5 make px4_sitl jmavsim
```

Ви можете застосувати коефіцієнт до всіх запусків SITL у поточній сесії за допомогою `EXPORT`:

```sh
export PX4_SIM_SPEED_FACTOR=2
make px4_sitl jmavsim
```

::: info
At some point IO or CPU will limit the speed that is possible on your machine and it will be slowed down "automatically".
Потужні комп'ютери зазвичай можуть запускати симуляцію зі швидкістю близько 6-10 разів, для ноутбуків досягається швидкість близько 3-4 разів.
:::

::: info To avoid PX4 detecting data link timeouts, increase the value of param [COM_DL_LOSS_T](../advanced_config/parameter_reference.md#COM_DL_LOSS_T) proportional to the simulation rate. Наприклад, якщо `COM_DL_LOSS_T` дорівнює 10 в реальному часі, при 10-кратному моделюванні швидкість збільшиться до 100.
:::

### Симуляція Lockstep

PX4 SITL і симулятори (jMAVSim або Gazebo Classic) налаштовані на роботу з кроком _lockstep_. Це означає, що PX4 і симулятор чекають один на одного для отримання повідомлень від датчиків і приводів, а не працюють зі своїми власними швидкостями.

::: info Lockstep makes it possible to [run the simulation faster or slower than realtime](#simulation_speed), and also to pause it in order to step through code.
:::

Послідовність кроків для lockstep наступна:

1. Симуляція надсилає повідомлення датчика [HIL_SENSOR](https://mavlink.io/en/messages/common.html#HIL_SENSOR) з міткою часу `time_usec` для оновлення стану та часу датчика PX4.
1. PX4 отримує це повідомлення, виконує одну ітерацію оцінювання стану, керування тощо і врешті-решт надсилає актуатору повідомлення [HIL_ACTUATOR_CONTROLS](https://mavlink.io/en/messages/common.html#HIL_ACTUATOR_CONTROLS).
1. Симуляція чекає, поки не отримає повідомлення від приводу/двигуна, потім моделює фізику і обчислює наступне повідомлення від датчика, яке знову надсилається до PX4.

Система починається з "вільного ходу", під час якого симуляція надсилає повідомлення від датчиків, зокрема про час, і, таким чином, запускає PX4, доки він не ініціалізується і не надішле відповідне повідомлення від приводу.

#### Вимкнення Lockstep симуляції

Lockstep симуляцію можна вимкнути, якщо, наприклад, SITL потрібно використовувати з тренажером, який не підтримує цю функцію. У цьому випадку симулятор і PX4 використовують системний час хоста і не чекають один на одного.

Щоб вимкнути lockstep у PX4, виконайте `make px4_sitl_default boardconfig` і встановіть символ `BOARD_NOLOCKSTEP` "Force disable lockstep", який знаходиться під панеллю інструментів.

Щоб вимкнути lockstep у Gazebo, відредагуйте [файл SDF моделі](https://github.com/PX4/PX4-SITL_gazebo-classic/blob/3062d287c322fabf1b41b8e33518eb449d4ac6ed/models/plane/plane.sdf#L449) і встановіть `<enable_lockstep>false</enable_lockstep>`.

Щоб вимкнути lockstep у jMAVSim, видаліть `-l` у [sitl_run.sh](https://github.com/PX4/PX4-Autopilot/blob/main/Tools/simulation/jsbsim/sitl_run.sh#L40) або іншим чином переконайтеся, що java-двійник запускається без прапора `-lockstep`.

<!-- Relevant lines in sitl_run.sh are: -->
<!-- # Start Java simulator -->
<!-- "$src_path"/Tools/simulation/jmavsim/jmavsim_run.sh -r 250 -l & SIM_PID=$! -->

### Сценарії запуску

Scripts are used to control which parameter settings to use or which modules to start. Вони знаходяться у каталозі [ROMFS/px4fmu_common/init.d-posix](https://github.com/PX4/PX4-Autopilot/tree/main/ROMFS/px4fmu_common/init.d-posix), файл `rcS` є основною точкою входу. Докладнішу інформацію наведено у розділі [Запуск системи](../concept/system_startup.md).

### Імітація збоїв та відмов датчиків/обладнання

[Імітація збоїв](../simulation/failsafes.md) пояснює, як викликати збої безпеки, такі як відмова GPS і розряд акумулятора.

## Середовище симуляції HITL

За допомогою симуляції Hardware-in-the-Loop (HITL) звичайна прошивка PX4 запускається на реальному обладнанні. Середовище моделювання HITL задокументовано: [HITL симуляція](../simulation/hitl.md).

## Інтеграція джойстиків/геймпада

_QGroundControl_ для ПК може підключатися до USB-джойстика/геймпада і надсилати команди руху та натискання кнопок на PX4 через MAVLink. Це працює як на SITL, так і на HITL симуляціях, і дозволяє вам безпосередньо керувати симульованим апаратом. Якщо у вас немає джойстика, ви можете керувати апаратом за допомогою екранних віртуальних паличок QGroundControl.

Для отримання інформації про налаштування див. _Посібник користувача QGroundControl_:

- [Налаштування пульта](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/joystick.html)
- [Віртуальний джойстик](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/settings_view/virtual_joystick.html)

<!-- FYI Airsim info on this setting up remote controls: https://github.com/Microsoft/AirSim/blob/master/docs/remote_controls.md -->

## Camera Simulation

PX4 підтримує захоплення як нерухомих зображень, так і відео з симулятора [Gazebo Classic](../sim_gazebo_classic/README.md). Це можна ввімкнути/налаштувати, як описано в розділі [Gazebo Glassic > Потокове відео](../sim_gazebo_classic/README.md#video-streaming).

Симуляція камери - це класичний gazebo плагін, який реалізує [MAVLink Camera Protocol](https://mavlink.io/en/protocol/camera.html). <!-- **PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/src/gazebo_geotagged_images_plugin.cpp -->. PX4 з'єднується/інтегрується з цією камерою _точно так само_, як і з будь-якою іншою камерою MAVLink:

1. [TRIG_INTERFACE](../advanced_config/parameter_reference.md#TRIG_INTERFACE) має бути встановлено у `3`, щоб налаштувати драйвер тригера камери для використання з камерою MAVLink :::tip У цьому режимі драйвер просто надсилає повідомлення [CAMERA_TRIGGER](https://mavlink.io/en/messages/common.html#CAMERA_TRIGGER) щоразу, коли запитується захоплення зображення. Для отримання додаткової інформації див. [Камера](../peripherals/camera.md).
:::
1. PX4 повинен перенаправляти всі команди камери між GCS і (симулятором) MAVLink Camera. Ви можете зробити це, запустивши [MAVLink](../modules/modules_communication.md#mavlink) з прапором `-f`, як показано на малюнку, вказавши UDP-порти для нового з'єднання.

   ```sh
   mavlink start -u 14558 -o 14530 -r 4000 -f -m camera
   ```

   ::: info
More than just the camera MAVLink messages will be forwarded, but the camera will ignore those that it doesn't consider relevant.
:::

Інші симулятори можуть використовувати такий самий підхід для реалізації підтримки камери.

## Запуск симуляції на віддаленому сервері

Симулятор можна запустити на одному комп'ютері, а доступ до нього отримати з іншого комп'ютера в тій же мережі (або в іншій мережі з відповідною маршрутизацією). Це може бути корисно, наприклад, якщо ви хочете протестувати програму для безпілотника, що працює на реальному комп'ютері-компаньйоні на фоні змодельованого транспортного засобу.

Це не працює "з коробки", оскільки PX4 за замовчуванням не маршрутизує пакети на зовнішні інтерфейси (щоб уникнути спаму в мережі та втручання різних симуляцій одна в одну). Замість цього він спрямовує трафік всередину - на "localhost".

Існує декілька способів зробити UDP-пакети доступними на зовнішніх інтерфейсах, як описано нижче.

### Використання MAVLink Router

[Mavlink-router](https://github.com/mavlink-router/mavlink-router) можна використовувати для маршрутизації пакетів з localhost на зовнішній інтерфейс.

Ви можете маршрутизувати пакети між SITL, запущеним на одному комп'ютері (що надсилає трафік MAVLink на localhost через UDP-порт 14550), і QGC, запущеним на іншому комп'ютері (наприклад, за адресою `10.73.41.30`):

- Запустіть _mavlink-router_ за допомогою наступної команди:

  ```sh
  mavlink-routerd -e 10.73.41.30:14550 127.0.0.1:14550
  ```

- Використовуйте конфігураційний файл _mavlink-router_.

  ```ini
  [UdpEndpoint QGC]
  Mode = Normal
  Address = 10.73.41.30
  Port = 14550

  [UdpEndpoint SIM]
  Mode = Eavesdropping
  Address = 127.0.0.1
  Port = 14550
  ```

::: info More information about _mavlink-router_ configuration can be found [here](https://github.com/mavlink-router/mavlink-router#running).
:::

### Увімкнення трансляції UDP

[Mavlink module](../modules/modules_communication.md#mavlink_usage) за замовчуванням маршрутизує до _localhost_, але ви можете увімкнути UDP-трансляцію за допомогою його опції `-p`. Будь-який віддалений комп'ютер у мережі може підключитися до симулятора, прослуховуючи відповідний порт (наприклад, 14550 для _QGroundControl_).

::: info UDP broadcasting provides a simple way to set up the connection when there is only one simulation running on the network. Не використовуйте цей підхід, якщо у мережі запущено декілька симуляцій (ви можете замість цього  [publish to a specific address](#enable-streaming-to-specific-address)).
:::

Це слід зробити у відповідному конфігураційному файлі, де викликається `mavlink start`. Наприклад: [/ROMFS/px4fmu_common/init.d-posix/px4-rc.mavlink](https://github.com/PX4/PX4-Autopilot/blob/main/ROMFS/px4fmu_common/init.d-posix/px4-rc.mavlink).

### Увімкнення стрімінгу на певну адресу

[Mavlink module](../modules/modules_communication.md#mavlink_usage) за замовчуванням спрямовує на _localhost_, але ви можете вказати зовнішню IP-адресу для потокового передавання за допомогою його параметра `-t`. Вказаний віддалений комп'ютер може підключитися до симулятора, прослуховуючи відповідний порт (наприклад, 14550 для _QGroundControl_).

Це слід зробити у різних конфігураційних файлах, де викликається `mavlink start`. Наприклад: [/ROMFS/px4fmu_common/init.d-posix/px4-rc.mavlink](https://github.com/PX4/PX4-Autopilot/blob/main/ROMFS/px4fmu_common/init.d-posix/px4-rc.mavlink).

### Тунелювання по SSH

Тунелювання SSH є гнучким варіантом, оскільки комп'ютер для моделювання та система, що його використовує, не обов'язково повинні знаходитися в одній мережі.

::: info
You might similarly use VPN to provide a tunnel to an external interface (on the same network or another network).
:::

Одним із способів створення тунелю є використання параметрів тунелювання SSH. Сам тунель можна створити, виконавши наступну команду на _localhost_, де `remote.local` - ім'я віддаленого комп'ютера:

```sh
ssh -C -fR 14551:localhost:14551 remote.local
```

UDP-пакети потрібно перетворити на TCP-пакети, щоб їх можна було перенаправляти через SSH. За допомогою утиліти [netcat](https://en.wikipedia.org/wiki/Netcat) можна працювати по обидва боки тунелю - спочатку для перетворення пакетів з UDP в TCP, а потім назад в UDP на іншому кінці.

:::tip QGC
must be running before executing _netcat_.
:::

На комп'ютері _QGroundControl_ трансляцію UDP-пакетів можна здійснити за допомогою наступних команд:

```sh
mkfifo /tmp/tcp2udp
netcat -lvp 14551 < /tmp/tcp2udp | netcat -u localhost 14550 > /tmp/tcp2udp
```

Команда на стороні симулятора тунелю SSH:

```sh
mkfifo /tmp/udp2tcp
netcat -lvup 14550 < /tmp/udp2tcp | netcat localhost 14551 > /tmp/udp2tcp
```

Номер порту `14550` дійсний для підключення до QGroundControl або іншої GCS, але повинен бути скоригований для інших кінцевих точок (наприклад, API розробника тощо).

Теоретично тунель може працювати нескінченно довго, але з'єднання _netcat_ доведеться перезапустити, якщо є якась проблема.

Скрипт [QGC_remote_connect.bash](https://raw.githubusercontent.com/ThunderFly-aerospace/sitl_gazebo/autogyro-sitl/scripts/QGC_remote_connect.bash) можна запустити на комп'ютері QGC для автоматичного налаштування/запуску вищевказаних інструкцій. Симуляція вже має бути запущена на віддаленому сервері, і ви повинні мати доступ по SSH до цього сервера.

[mav_mode_flag_hil_enabled]: https://mavlink.io/en/messages/common.html#MAV_MODE_FLAG_HIL_ENABLED
[hil_actuator_controls]: https://mavlink.io/en/messages/common.html#HIL_ACTUATOR_CONTROLS
[hil_sensor]: https://mavlink.io/en/messages/common.html#HIL_SENSOR
[hil_gps]: https://mavlink.io/en/messages/common.html#HIL_GPS
[hil_optical_flow]: https://mavlink.io/en/messages/common.html#HIL_OPTICAL_FLOW
[hil_state_quaternion]: https://mavlink.io/en/messages/common.html#HIL_STATE_QUATERNION
[hil_rc_inputs_raw]: https://mavlink.io/en/messages/common.html#HIL_RC_INPUTS_RAW