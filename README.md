# Caddy AI2

Repositorio general del proyecto **Caddy AI2** (robot agrícola, proyecto CERVAREC). Reúne bajo un único punto de entrada la **simulación** y el **vehículo real**; cada parte vive en su propio repositorio y se incluye aquí como submódulo git.

**ROS 2:** Jazzy | **Plataforma:** Ubuntu 24.04 | **Simulador:** Gazebo Harmonic

Tablero del proyecto: https://github.com/users/racarla96/projects/7

---

## Estructura

| Submódulo | Repositorio | Rama | Contenido |
|---|---|---|---|
| [`sim/`](sim) | [`caddy_ai2_ros2_sim`](https://github.com/racarla96/caddy_ai2_ros2_sim) | `jazzy` | Simulación en Gazebo Harmonic: launch principal, mundos, sensores simulados, localización y navegación |
| [`robot/`](robot) | [`caddy_ai2_ros2_robot`](https://github.com/racarla96/caddy_ai2_ros2_robot) | `jazzy` | Vehículo real: launch del sistema completo, drivers CAN, sensores y configuración |

Cada uno es a su vez un repositorio de integración que agrupa sus paquetes ROS 2 (descripción, drivers, controladores, etc.). Consulta el README de cada submódulo para la lista de paquetes y las instrucciones de uso.

Los paquetes compartidos (p. ej. `caddy_ai2_ros2_description`, adaptadores bicycle→ackermann, `robot_description_publisher`) viven en repositorios propios y los consumen tanto `sim` como `robot`, de modo que el mismo código sirve para simulación y vehículo real.

---

## Clonar

```bash
git clone --recurse-submodules git@github.com:racarla96/caddy_ai2.git
cd caddy_ai2
```

Si ya lo clonaste sin `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

Solo una de las partes:

```bash
git submodule update --init --recursive sim     # o: robot
```

---

## Actualizar

Los submódulos siguen la rama indicada en `.gitmodules`. Para traer lo último de cada uno y registrar la nueva versión en este repo:

```bash
git submodule update --remote --recursive
git add sim robot
git commit -m "Update submodules"
```

Este repo fija un commit concreto de `sim` y `robot`: un commit aquí describe un par de versiones que funcionan juntas.

---

## Uso

- **Simulación:** ver [`sim/README.md`](sim/README.md).
- **Vehículo real:** ver [`robot/README.md`](robot/README.md).
