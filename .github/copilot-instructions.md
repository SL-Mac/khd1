## 目的
为代码生成/编辑 AI（例如 Copilot/Agent）提供即时可用的、与本仓库相关的上下文与规则，帮助自动化修改时保持项目结构、构建与 CubeMX 约定不被破坏。

## 大体架构（快速理解）
- MCU: STM32F103C8T6（见 `khd1.ioc` 中 `Mcu.DeviceId`）。
- 代码基于 STM32CubeMX + HAL。生成的 CMake 集成在 `cmake/stm32cubemx`，用户代码主要放在 `Core/Src` 与 `Core/Inc`。
- 外围/库代码放在 `Lib/`（例如 `Lib/oled/`、`Lib/nrf24l01/`），并在顶层 `CMakeLists.txt` 中通过 `target_sources` / `target_include_directories` 引入。

## 关键工作流（可直接运行）
- 配置/构建（在 PowerShell）：
  - cmake --preset Debug
  - cmake --build --preset Debug
  说明：项目使用 Ninja（见 `CMakePresets.json`），并通过 `cmake/gcc-arm-none-eabi.cmake` 指定交叉工具链。
- 智能感知：项目启用了 compile_commands（见 `build/Debug/compile_commands.json`），可直接用于 clangd/静态分析。

## 项目约定与要点（不要违背）
- CubeMX 约定：自动生成/手工代码的边界由 `/* USER CODE BEGIN */` / `/* USER CODE END */` 标记。修改时尽量把自定义逻辑放入这些区块，避免被 CubeMX 覆写（参见 `Core/Src/main.c`）。
- 外设初始化函数命名约定：MX_GPIO_Init、MX_DMA_Init、MX_I2C1_Init、MX_TIM3_Init 等 —— 在新增外设或调用时遵循相同风格。
- 添加/注册源码：将新驱动或模块放到 `Lib/` 或 `Core/Src`，并在顶层 `CMakeLists.txt` 的 `target_sources(${CMAKE_PROJECT_NAME} PRIVATE ...)` 中加入相对路径。

## 集成点与依赖
- HAL 与 CMSIS 位于 `Drivers/STM32F1xx_HAL_Driver` 与 `Drivers/CMSIS`。
- 设备配置由 `khd1.ioc` 管理（CubeMX 项目文件）。更改外设配置后，注意同步或合并 `cmake/stm32cubemx` 下生成的文件。
- 外部库（如 OLED、NRF24）通过 `Lib/*` 包含并在顶层 CMake 中列出，修改这些库时同时检查 `target_include_directories` 与 `target_sources`。

## 代码示例（便于自动化修改）
- 安全修改示例：在 `Core/Src/main.c` 的 `/* USER CODE BEGIN 2 */` 中添加初始化代码（比如 HAL 外设启动），示例已有 `HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1);`。
- 新源文件示例：添加 `Core/Src/foo.c`，并在 `CMakeLists.txt` 的 `target_sources(... PRIVATE ...)` 列表中添加 `Core/Src/foo.c`。

## 风险与注意事项
- 避免直接改写 `cmake/stm32cubemx` 生成的代码，除非确认不会被 CubeMX 重新生成覆盖。
- 不要改变 `CMakePresets.json` 的工具链路径结构，除非用户也修改了工具链或在文档中说明了替代方案。

## 需要人工确认的场景（Agent 遇到这些时应询问）
- 当修改影响 `khd1.ioc` 或 `cmake/stm32cubemx` 生成文件时，先询问是否允许更改 CubeMX 配置或重新生成。
- 添加新的交叉编译工具链或更改 `cmake/gcc-arm-none-eabi.cmake` 时，询问用户确认工具链路径与版本。

如需调整或补充项目内特定约定（例如自定义 flashing/debug 脚本位置），请告知我需要补充的具体文件或常用工具链/调试器。谢谢！
