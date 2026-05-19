# JACTECH 2024 — Robot Code

Código do robô da equipe JACTECH para a temporada FRC 2024 (_Crescendo_). O projeto segue a arquitetura **Command-Based** do WPILib e integra múltiplas bibliotecas de fornecedores.

---

## Imagem do Robô

<img width="1125" height="728" alt="Imagem Robo" src="https://github.com/user-attachments/assets/e3de6cab-5040-4287-a4fe-c8a3801e6781" />
<img width="998" height="630" alt="image" src="https://github.com/user-attachments/assets/1d5b7567-4294-484e-a886-b8bd57b2f136" />

## Imagem do Statibotics

<img width="1402" height="410" alt="image" src="https://github.com/user-attachments/assets/ce8224f7-8b63-4e34-8ad5-ee45651625e4" />

## Stack Tecnológica

| Item | Versão |
|---|---|
| Java | 17 |
| WPILib | 2024.3.2 |
| Gradle (FRC Toolchain) | GradleRIO |
| PathPlanner | Integrado via vendor |
| Phoenix 6 (CTRE) | Integrado via vendor |
| PhotonVision | Integrado via vendor |
| REV Robotics | Integrado via vendor |

---

## Estrutura do Projeto

```
src/main/java/
├── frc/robot/
│   ├── Main.java                    — Ponto de entrada
│   ├── Robot.java                   — Scheduler principal
│   ├── RobotContainer.java          — Mapeamento de botões e comandos
│   ├── Constants.java               — Todas as constantes de hardware e controle
│   ├── subsystems/
│   │   ├── DriveTrain/              — Swerve drive
│   │   ├── Launcher/                — Lançador de notes
│   │   ├── JointLauncher/           — Angulação do lançador
│   │   ├── Elevator/                — Elevator (angulação + movimentação)
│   │   ├── Climber/                 — Climber esquerdo e direito
│   │   ├── Led/                     — Controle de LEDs (REV Blinkin)
│   │   └── Vision/                  — Estimativa de pose com PhotonVision
│   ├── commands/                    — Lógica de controle por subsistema
│   └── auto/                        — Rotinas autônomas
└── frc/JacLib/
    ├── JoystickOI.java              — Mapeamento de botões dos joysticks
    ├── FieldConstants.java          — Dimensões do campo
    ├── utils/
    │   ├── PhotonLL.java            — Wrapper da câmera PhotonVision
    │   └── SwerveUtils.java         — Utilitários de swerve
    └── imu/
        └── ImuSubsystem.java        — Subsistema do IMU
```

---

## Subsistemas

### Drivetrain — Swerve Drive

- **Módulos:** 4x REV MAXSwerve
- **Motores de translação (CAN):** 1 (FL), 2 (FR), 3 RL), 4 (RR)
- **Motores de rotação (CAN):** 10 (FL), 20 (FR), 30 (RL), 40 (RR)
- **Giroscópio:** Pigeon 2 (CAN 8)
- **Pinion:** 13 dentes — diâmetro da roda: 0,07285 m
- **Velocidade máxima:** 3 m/s translação, 1,5π rad/s rotação
- **Slew rate:** 6 rad/s (direção), 4 %/s (magnitude e rotação)
- **Modos de controle:** Field-relative e robot-relative
- **Integração:** PathPlanner AutoBuilder com configuração holonômica

**PID interno dos módulos:**

| Controlador | kP | kI | kD | FF |
|---|---|---|---|---|
| Drive | 0.1 | 0 | 0 | 1/FreeSpeedRps |
| Turning | 0.5 | 0 | 0 | 0 |

---

### Launcher — Lançador de Notes

- **Motor de lançamento:** CANSparkFlex (CAN 24) — velocidade máxima: 100%
- **Motor de trigger:** CANSparkMax (CAN 23) — velocidade: 100% (lançamento) / 80% (intake)
- **Detecção de note:** Sensor de corrente — threshold 40 A
- **Estados:** `Launch`, `Intake`, `Trigger`, `TriggerAmp`, `Static`

---

### Joint Launcher — Angulação do Lançador

- **Motores:** 2x CANSparkFlex — esquerdo mestre (CAN 21), direito seguidor invertido (CAN 22)
- **Controle:** PID por encoder relativo
- **Calibração:** `definePosition()` zera os encoders na posição atual

**PID:**

| kP | kI | kD |
|---|---|---|
| 0.1 | 0 | 0 |

**Setpoints de posição (rotações):**

| Posição | Valor |
|---|---|
| Home | -5.0 |
| Subwoofer | -37.66 |
| Podium | -28.0 |
| Floor Intake | -5.3 |
| Amp | -1.97 |

---

### Elevator

Sistema de dois eixos independentes.

#### Elevator Angle — Angulação

- **Motores:** 2x CANSparkMax — direito mestre (CAN 11), esquerdo seguidor invertido (CAN 12)
- **Range:** -0.05 a -1.9 (unidades normalizadas)
- **Velocidade dinâmica:** 0.15 a 0.35 dependendo da posição

**PID:**

| kP | kI | kD |
|---|---|---|
| 1.85 | 0 | 0 |

#### Elevator Move — Movimentação Vertical

- **Motor:** CANSparkMax (CAN 13)
- **Range:** 0 a 165 unidades de encoder
- **Velocidade máxima:** 0.5 — limite de corrente: 35 A

**PID:**

| kP | kI | kD |
|---|---|---|
| 0.2 | 0 | 0 |

**Setpoints do Elevator (ângulo / altura):**

| Posição | Ângulo | Altura |
|---|---|---|
| Home | -0.1 | 0 |
| Floor Intake | -1.66 | 0 |
| Amp | -0.4 | 145 |
| Subwoofer | -1.0 / -0.80 | 0 |
| Podium | -0.67 | 0 |

---

### Climber — Gancho de Escalada

- **Motor esquerdo:** CANSparkFlex (CAN 31)
- **Motor direito:** CANSparkFlex (CAN 32)
- **Limite de velocidade:** ±1.8
- **Limite de posição esquerdo:** 0 a -227
- **Limite de posição direito:** 0 a -245

**PID (compartilhado via `ClimberConstants`):**

| kP | kI | kD |
|---|---|---|
| 0.2 | 0 | 0 |

---

### Sistema de Visão

Dois pipelines de câmera rodando em threads separadas via `Notifier` a cada 20 ms.

#### Câmeras

| Câmera | Nome CAN | Posição (x, y, z) | Rotação |
|---|---|---|---|
| Frontal | `Camera_Module_v1` | 325 mm, 300 mm, 160 mm | Pitch 325° |
| Lateral esquerda | `Camera_Left` | -315 mm, 300 mm, 160 mm | Roll 0°, Pitch 30°, Yaw 90° |

#### Estimativa de Pose

- **Filtro:** `SwerveDrivePoseEstimator` (Kalman Filter)
- **Estratégia:** Multi-tag PNP com fallback para menor ambiguidade
- **Layout:** AprilTag 2024 Crescendo
- **Threshold de ambiguidade:** 0.2

**Desvios padrão:**

| | x (m) | y (m) | θ (graus) |
|---|---|---|---|
| Estado | 0.05 | 0.05 | 5 |
| Visão | 0.1 | 0.1 | 10 |

---

### LEDs

- **Controlador:** REV Blinkin (PWM porta 1)
- **Estados:** Rainbow, Green, Blue, Orange, entre outros

---

## Autônomo

Integrado com **PathPlanner**. Comandos nomeados registrados:

| Nome | Função |
|---|---|
| `Floor Intake` | Posicionar para coletar do chão |
| `Subwoofer launch` | Posicionar e lançar no subwoofer |
| `Amp Launching` | Posicionar e lançar no amp |
| `Podium Launching` | Posicionar e lançar no podium |
| `Home` | Retornar à posição padrão |
| `Launching` | Sequência de lançamento |
| `launchingReturn` | Retorno pós-lançamento |
| `SubwooferReturn` | Retorno do subwoofer |

Rotinas disponíveis: `Reto` (linha reta de teste), `WorldChampion`.

---

## Mapeamento de Controles

### Driver (porta 0)

| Controle | Ação |
|---|---|
| Analógico esquerdo | Translação (X/Y) |
| Analógico direito | Rotação |
| START | Zerar heading (giroscópio) |
| B | Virar para o speaker |
| X | Alinhar ao AprilTag |
| LB | Climber desce |
| RB | Climber sobe |
| L3 / R3 | Calibrar joint launcher / LEDs |

### Operador (porta 1)

| Controle | Ação |
|---|---|
| RT | Lançar note |
| LT | Coletar note (intake) |
| START | Trigger ativo |
| BACK | Trigger amp |
| Y | Posição Home |
| POV Cima | Posição Subwoofer |
| POV Baixo | Posição Floor Intake |
| POV Esquerda | Posição Amp |
| POV Direita | Posição Podium |
| LB / RB | Ajuste fino da angulação do lançador (±0.1/s) |
| X / B | Ajuste da angulação do elevator |
| A | Definir posição zero (calibrar encoders) |

---

## Build e Deploy

```bash
# Build do projeto
./gradlew build

# Deploy para o robô (requer conexão)
./gradlew deploy

# Simulação
./gradlew simulateJava
```

---

## Identificadores CAN

| ID | Dispositivo |
|---|---|
| 1 | Drive FL |
| 2 | Drive FR |
| 3 | Drive RL |
| 4 | Drive RR |
| 8 | Pigeon 2 (Gyro) |
| 10 | Turning FL |
| 11 | Elevator Angle Right |
| 12 | Elevator Angle Left |
| 13 | Elevator Move |
| 20 | Turning FR |
| 21 | Joint Launcher Left |
| 22 | Joint Launcher Right |
| 23 | Trigger Motor |
| 24 | Launcher Motor |
| 30 | Turning RL |
| 31 | Climber Left |
| 32 | Climber Right |
| 40 | Turning RR |
