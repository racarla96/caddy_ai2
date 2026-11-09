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
| `sensors_rs_helios_5515`, `camera_zed_2i`, `imu_vectornav_vn100` | ✓ | ✗ | Integrar en `robot` (ZED en Jetson/contenedor) |
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
- [ ] **Variantes del vehículo (a decidir):** con PLC y sin PLC. Propuesta: argumento `use_plc` (o `config/with_plc/` y `config/no_plc/`). Tiene que caber el bicycle controller, el orquestador y el watchdog.

### Fase C — `robot` como workspace repo

**Decisión pendiente:** `sim` usa submódulos y `robot` usa `.repos`. Propuesta: unificar en **submódulos** (el paraguas fija las versiones y no hay dos mecanismos), como ya preveía el plan de junio.

- [ ] Pasar `robot` a submódulos en `src/`; paquete de integración en `src/caddy_ai2_ros2_robot/`; `.gitignore` de `build/ install/ log/`.
- [ ] `robot.launch.py`: `robot_state_publisher`, `controller_manager`, controladores, sensores, ABS. Argumentos: `namespace`, `prefix`, `use_rviz`, `use_abs`, `use_plc`, `use_joy`.
- [ ] `controllers.yaml.j2` unificado; calibración de dirección (`counts_per_radian`, `abs_zero_position`, `gain_inc_to_abs`) en un fichero de calibración dedicado.
- [ ] Mover configuración de controladores de los drivers a `robot` y `sim`.
- [ ] Eliminar `bringup/` y `description/urdf/` standalone de los drivers (dejar solo el fragmento `ros2_control`).

### Fase D — Simulación (MVSim)

- [ ] Decidir si `sim` incluye `navsat_generic`.
- [ ] Alinear el chasis visual (offset) y centralizar la corrección en `description`.
- [ ] Parámetros físicos reales: centro de gravedad, masa, distribución de pesos; medir poses reales de las ZED.
- [ ] Paridad sim ↔ robot: mismos topics y frames; smoke test (publicar referencia, comprobar `joint_states`).
- [ ] Arreglar EasyNav en Gazebo: **aparcado** con la descontinuación.

### Fase E — Cámaras en hardware real

- [ ] Contenedor Docker para el Jetson Orin NX / ZED Box (repo orquestador previsto `caddy_ai2_ros2_zedbox_sensors`).
- [ ] Reconciliar los frames ópticos del wrapper real con los `<frame_id>_optical` del URDF.
- [ ] Probar `zed_2i.launch.py` con `serial_number` en las dos cámaras.

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
B (URDF robot real) → C (robot como workspace) → G (validación)
        ↓
D, E, F, H en paralelo según haya hardware disponible
```

## 6. Decisiones abiertas

1. ¿`robot` con submódulos (propuesto) o `.repos`?
2. ¿Variante con/sin PLC por argumento de launch o por carpeta de configuración?
3. ¿`siyi_mk15`, `control_watchdog`, `teleop_radio_esp32_bridge` y `control_system_bicycle_controller` entran en el sistema o se archivan?
4. ¿`sim` incluye `navsat_generic`?
5. ¿`caddy_ai2_ros2_navigation` y `localization` entran también en `robot`?
