# Plan de reorganización — Caddy AI2

Estado: 2026-11-09. Sustituye a `PLAN_robot_real.md` y `PLAN_sim_nav2.md` de `caddy_ai2_ros2_conventions`
(de junio, desfasados: hablan de Nav2 y de Gazebo como simulador principal).

## 1. Decisiones tomadas

| Tema | Decisión |
|---|---|
| Estructura | Repo paraguas `caddy_ai2` → submódulos `sim/` y `robot/`; cada uno agrupa sus paquetes. Un repo por paquete. |
| Simulador | **MVSim** es el principal. **Gazebo se descontinúa por ahora** (se mantiene el código, sin evolucionar). |
| Navegación | **EasyNav** (A* sobre NavMap + Regulated Pure Pursuit), no Nav2. |
| Cámaras | 2 ZED 2i, dos disposiciones (A: dos delanteras; B: delantera y trasera). Verificadas en MVSim; ya en `jazzy`. |
| Descripción | Fuente única `caddy_ai2_ros2_description/bringup/config/robot_params.yaml` (+ `sensor_params.yaml` por sensor, Jinja2), compartida por sim y robot. |
| Nombres | Prefijo `caddy_ai2_ros2_`; el nombre del paquete = nombre del repo. Licencia CC BY 4.0 por defecto. |
| Material histórico | `caddy_ai2_extras` (privado), sin mantenimiento. `ros1_caddy_ai2_robotnik_sensors` se conserva. Los forks de Robotnik se eliminaron. |

## 2. Hecho

- [x] Trabajo pendiente de los workspaces antiguos subido; workspaces antiguos borrados.
- [x] `caddy_ai2_ros2_sensors_navsat_generic` con repo propio (privado).
- [x] `caddy_ai2_extras` con el sketch del tacómetro.
- [x] Rama `feat/zed-2i-cameras` fusionada en `jazzy` (`description`, `gazebo_simulation`, `sim`).
- [x] Vídeos de las dos disposiciones en MVSim (laboratorio UPV).
- [x] Paquete SICK renombrado a `caddy_ai2_ros2_sensors_sick_lms_291` y referencias actualizadas.
- [x] `robot`: nombres corregidos y `.repos` completo (11 repos).
- [x] Punteros de `sim` y del paraguas actualizados.
- [x] `robot` dividido en `caddy_ai2_ros2_robot_lowlevel` (LattePanda) y `caddy_ai2_ros2_robot_perception` (Jetson), ambos privados y compilando desde un clon limpio; `robot` es ahora su paraguas.

## 3. Estado de los paquetes

| Paquete | Sim | Robot | Pendiente |
|---|---|---|---|
| `description` | ✓ | ✓ | URDF/SDF del robot real (fase B), mover meshes duplicados de `mvsim_simulation` |
| `bicycle_to_ackermann_{steering,traction}_adapter` | ✓ | ✓ | Metadatos (`description`, `license`) |
| `robot_description_publisher` | ✓ | ✓ | — |
| `control_system_steering_driver` | — | ✓ | Migrar xacro → `URDF.j2`, README, release (ver A.1) |
| `control_system_traction_driver` | — | ✓ | Ídem + TODOs del driver (signo RPM, `prefix`, temporización) (A.2) |
| Sensores SICK, YDLidar, SBG | ✓ | ✓ | Validar IMU NED→ENU en real; SBG da también GPS (desactivar `navsat_generic` en real) |
| `sensors_navsat_generic` | ✗ | ✓ | Decidir si `sim` lo necesita |
| `sensors_rs_helios_5515`, `camera_zed_2i` | ✓ | ✗ | Integrar en `robot_perception` (Jetson, contenedor) |
| `imu_vectornav_vn100` | ✓ | ✓ (perception) | Instalar en la Jetson |
| GPS u-blox | ✗ | ✗ | Comprar (Mouser), crear paquete (Jetson) |
| `localization`, `navigation` | ✓ | ✗ | Integrar en `robot` cuando haya robot real autónomo |
| `automatic_brake_system` | — | ✗ | Revisar en Jazzy, README, release (A.3) |
| `plc_schneider_tm241ce40t` | — | ✗ | Driver + simulador de PLC y luces (A.4) |
| `controllers` | — | ✗ | Revisar qué centraliza (A.5) |
| `siyi_mk15` | — | ✗ | Decidir si forma parte del sistema o es un extra |
| `conventions` | — | — | Reescribir PLAN/TASKS/ARCHITECTURE con esta decisión |
| `common` | — | ✓ | — |
| `control_system_bicycle_controller`, `control_watchdog`, `teleop_radio_esp32_bridge` | — | ✗ | Decidir: integrar o archivar |

## 4. Fases

### Fase A — Preparar los paquetes de hardware

- [ ] **A.1 steering_driver:** `description/hardware.urdf.j2` en lugar de xacro; `bringup/config/` → `config/`; `.gitignore`; README (CiA 402, parámetros, bus CAN); release v1.0.0. La rama `jazzy-dev` tiene tests y calibración EPC: fusionar en `jazzy`.
- [ ] **A.2 traction_driver:** lo mismo (Curtis CAN). TODOs: verificar signo RPM, parámetro `prefix` multi-robot, parámetros de temporización como en steering, nombre del script (`setup_can_trac_drv.sh`).
- [ ] **A.3 automatic_brake_system:** probar en Jazzy / Python 3.12, `.gitignore`, README, release.
- [ ] **A.4 plc_schneider_tm241ce40t:** driver Modbus TCP, simulador de PLC/luces, topics y servicios, panel RViz/rqt, README, release.
- [ ] **A.5 controllers:** documentar qué centraliza; decidir si se funde con los drivers.
- [ ] **A.6 Metadatos:** `description` y `license` en `package.xml` de adaptadores, `gazebo_simulation` y `description`.
- [ ] **Validación hacia adelante:** probar con el bicycle controller en el robot real antes de seguir.

### Fase B — Descripción del robot real

- [ ] URDF para hardware real que extienda el modelo base e inyecte los `hardware.urdf.j2` de steering y traction y los sensores (sin Gazebo).
- [ ] `display.launch.py` con argumento `mode: sim|robot`.
- [x] **Variantes:** no hay variante sin PLC; el vehículo real siempre lleva PLC. Tiene que caber el bicycle controller, el orquestador y el watchdog.

### Fase C — `robot` dividido en dos subrepos (por ordenador)

El vehículo tiene dos ordenadores, así que `robot` deja de ser un único workspace y pasa a ser un paraguas con dos subrepos:

| Subrepo (nombre propuesto) | Ordenador | Contenido |
|---|---|---|
| `caddy_ai2_ros2_robot_lowlevel` | **LattePanda Alpha** (control de bajo nivel) | `steering_driver`, `traction_driver`, SICK LMS 291, PLC Schneider, SBG IG-500N (IMU), `common`, adaptadores bicycle→ackermann, controladores, ABS |
| `caddy_ai2_ros2_robot_perception` (antes previsto `caddy_ai2_ros2_zedbox_sensors`) | **Jetson Orin NX** (percepción, en contenedor Docker) | GPS u-blox propio, lidar 3D (Helios RS-5515), las dos ZED 2i |

- [x] Crear los dos subrepos, cada uno como workspace con submódulos en `src/` (mismo patrón que `sim`), y `caddy_ai2_ros2_robot` pasa a ser el paraguas de ambos.
- [ ] Cada subrepo con su `robot.launch.py` propio y su `package.xml`. El de percepción en un contenedor (imágenes en Docker Hub, `racarla96`).
- [ ] Nuevo paquete `caddy_ai2_ros2_sensors_gps_ublox` (hoy no existe repo): driver u-blox + fragmento de descripción.
- [x] **Posición/GPS:** hay **tres fuentes** que conviven, cada una con su topic: el GPS del SBG IG-500N (LattePanda), el u-blox (Jetson) y el GNSS integrado de la **ZED Box** (la Jetson Orin NX es una ZED Box de Stereolabs). Qué fuente(s) entran en el EKF de `localization` se decide al probarlas en el vehículo. `navsat_generic` (solo simulación) queda desactivado en real.
- [x] **Red:** los dos ordenadores están en la misma red, así que no hace falta zenoh: mismo `ROS_DOMAIN_ID` y descubrimiento DDS. Pendiente: fijar el RMW y la interfaz de red (por ejemplo CycloneDDS con `NetworkInterface` explícita), limitar qué información cruza entre los dos equipos (los topics pesados, nube Helios e imágenes ZED, no tienen por qué salir de la Jetson salvo los que consuma el LattePanda), que los topics pesados (nube Helios, imágenes ZED) no saturen el enlace, y **sincronizar la hora** entre los dos equipos (chrony), imprescindible para TF y fusión de sensores.
- [ ] `robot_description` único: el URDF del robot real se renderiza igual en ambos ordenadores; cada uno lanza solo los sensores que tiene (`robot_params.yaml` con un campo `host: lattepanda|jetson`).
- [x] Migrar de `.repos` a submódulos (hecho en ambos subrepos).
- [ ] `controllers.yaml.j2` unificado y calibración de dirección en un fichero dedicado (solo LattePanda).
- [ ] Mover configuración de controladores de los drivers a `lowlevel` y `sim`; eliminar `bringup/` y `description/urdf/` standalone de los drivers.

### Fase D — Simulación (MVSim)

- [ ] Decidir si `sim` incluye `navsat_generic`.
- [ ] Alinear el chasis visual (offset) y centralizar la corrección en `description`.
- [ ] Parámetros físicos reales: centro de gravedad, masa, distribución de pesos; medir poses reales de las ZED.
- [ ] Paridad sim ↔ robot: mismos topics y frames; smoke test (publicar referencia, comprobar `joint_states`).
- [ ] Arreglar EasyNav en Gazebo: **aparcado** con la descontinuación.

### Fase E — Jetson Orin NX y sensores de percepción

- [ ] Actualizar el SO de la Jetson a **Ubuntu 24.04** con ROS 2 Jazzy. La Jetson es una **ZED Box Orin NX** de Stereolabs (viene con JetPack y GNSS de fábrica): comprobar con Stereolabs qué versiones de SO admite y si reflashear afecta a la garantía o al GNSS integrado. Antes (junio) el plan era dejar JetPack 6.2 / Ubuntu 22.04 y usar un contenedor con Jazzy; verificar que existe un JetPack/L4T para Orin NX con Ubuntu 24.04 y que el ZED SDK y los drivers lo soportan antes de reflashear. Si no, mantener el contenedor.
- [ ] Instalar drivers y paquetes ROS 2 de: **Robosense Helios 5515 (RS-H32F70)** (lidar 3D), **VectorNav VN-100** (IMU) y **ZED 2i** (dos cámaras). El VN-100 va en la Jetson (`robot_perception`).
- [ ] Contenedor Docker o instalación nativa según el punto anterior (imágenes en Docker Hub, `racarla96`).
- [ ] **Demo de las cámaras en la simulación** (cuando estén integradas en la simulación del laboratorio) y, a partir de ella, planificar cómo montarlas en el vehículo real (disposición A o B, medidas de la barra).
- [ ] Reconciliar los frames ópticos del wrapper real con los `<frame_id>_optical` del URDF.
- [ ] Probar `zed_2i.launch.py` con `serial_number` en las dos cámaras.
- [ ] **Compra en Mouser:** GPS u-blox y lo que haga falta de paso (antena, cable, conversor serie/USB, alimentación; revisar lista tras elegir el modelo).

### Fase F — Seguridad y puesta en marcha

- [ ] Parada de emergencia (`/e_stop`) y estado seguro al perder el CAN.
- [ ] `scripts/check_hardware.sh` (CAN, IMU, LiDAR); procedimiento de arranque; sudoers para CAN; evaluar RT-PREEMPT.

### Fase G — Validación en el robot

- [ ] `ros2 control list_hardware_interfaces` / `list_controllers` correctos; `/joint_states`, `/imu`, scans y TF completos.
- [ ] Avance recto con referencia 0,3 m/s; el ABS responde al obstáculo.

### Fase H — Documentación y gobierno

- [ ] Reescribir `conventions` (PLAN, TASKS, ARCHITECTURE) para reflejar MVSim + EasyNav; marcar los antiguos como obsoletos.
- [ ] Renombrar el paquete de navegación si procede (`caddy_ai2_ros2_navigation` ya usa EasyNav; basta documentarlo).
- [ ] README de `description` para multirepo.
- [ ] Release v1.0.0 de cada paquete al estabilizarse; issues del proyecto 7 apuntando a cada repo.

## 5. Orden propuesto

```
Compilar y probar todo (sim + robot)          ← siguiente paso
        ↓
A.6 metadatos  ‖  A.1/A.2 drivers  ‖  A.3/A.4/A.5
        ↓
Validación HW (bicycle controller)
        ↓
B (URDF robot real) → C (robot en dos subrepos: lowlevel + perception) → G (validación)
        ↓
D, E, F, H en paralelo según haya hardware disponible
```

## 6. Decisiones abiertas

1. ~~Submódulos o `.repos` en `robot`~~: resuelto, submódulos.
2. ~~Variante con/sin PLC~~: resuelto, siempre con PLC.
3. ¿`siyi_mk15`, `control_watchdog`, `teleop_radio_esp32_bridge` y `control_system_bicycle_controller` entran en el sistema o se archivan?
4. ¿`sim` incluye `navsat_generic`?
5. ¿`caddy_ai2_ros2_navigation` y `localization` entran también en `robot`?
