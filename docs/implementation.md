# Snip2Pin 实现与结构

## 需求理解与实现

- 实现前先弄清用户要解决什么问题、为什么需要这个行为，以及怎样才算使用顺畅。区分用户的目标与用户提出的具体做法，不能把具体做法当成需求的全部；实现和验证都要围绕目标展开。
- 新方案或评审建议与用户先前指令、已确认需求冲突时，说明冲突、依据与推荐方案，向用户确认；确认前不实施冲突部分，其他独立工作继续。不能擅自覆盖旧要求，也不能仅因旧要求而默认否决新方案。
- PR 标题保持简短；正文和进度评论只列出问题列表、修复列表，面向不了解内部沟通的贡献者，不展开调试过程或堆叠实现细节。技术评审发现放在对应 PR 评论中；仓库文档只保留当前使用方法、开发命令、架构与长期维护规范。
- 公开引用 commit、PR、Issue、CI 运行或代码位置时使用可点击链接，便于追溯；commit 用短 SHA 作链接文字并指向完整 SHA，PR／Issue 使用编号作链接文字。发布前确认链接目标真实存在。
- 不得将聊天内容、指令原文、个人信息、偏好与授权经过、本地调试细节、操作／安装记录、内部协作过程、agent 角色或模型配置写入公开 PR、Issue、提交说明等。公开内容须先提炼为必要的项目技术信息，不直接复制对话或内部评审报告；验收仅记录提交、平台、场景和技术结论。
- 方案必须考虑通用性和可维护性，覆盖不同平台、显示器布局、DPI、缩放比例、图片尺寸和边缘位置。把共同规则封装在合适的模块中，平台差异留在适配层；保持项目简洁，不为假设中的需求增加抽象或功能。
- 禁止通过硬编码位置、偏移量或针对某个场景的特殊数值修补现象。坐标和布局应从实际边界及统一的坐标换算推导，样式参数和阈值应有明确含义并集中管理。仅把散落数字改成常量，不能代替对模型和计算方式的改进。
- 出现偏移或位置异常时，先定位坐标单位、DPI、缩放、窗口边框、原生系统约束及生命周期中的根因。不得用反复移动、缩放、重试或轮询来掩盖错误；之前用定位重试补偿偏移既未解决根因，又引入了死循环问题，不能再采用这种做法。
- 图标放到图片下方的目的是让用户操作更顺畅：图片移动时复制和关闭图标仍应留在屏幕内并可操作。“顶部时把图标放下面”只是实现手段；macOS 顶部应让完整图片贴紧菜单栏下缘，不能被旧的透明上边距阻挡，也不能用裁切实现。其他方向应保留图片越界移动能力。
- 验证用户最终能否完成操作，而不只验证中间数值或某个分支。涉及窗口行为时，检查真实的平台约束和完整交互路径；布局计算正确或隐藏窗口能设置位置，不代表用户拖动时一定正确。明确区分自动化验证与用户实际交互验证，发现缺口时补足测试。
- 确认原始问题真正消失前不得合并或启用自动合并，证据不足的修复保持 Draft PR。闪烁、跳动、瞬时偏移须验证可见原生窗口连续操作的中间帧；自动检查、网页或隐藏缓冲区测试通过不能视为真实交互通过。用户不允许操控屏幕时先安装验证构建，等待用户明确确认；仍能复现就继续修复，不先合入或宣称完成。验收记录关联具体提交，后续影响该行为的改动须重新验证。
- 原生窗口是当前几何的唯一权威；Rust 保留不可变原图、初始放置与图内按钮布局。鼠标／触屏原生拖动及边缘缩放不自行求解，不保留重复期望 Rect 或反馈补偿；显式键盘、滚轮和触摸板命令可根据实际几何计算一次调整。跨 DPI 先统一坐标单位。
- macOS 原生拖动使用所属窗口的原始鼠标按下事件，不能在延迟重绘后用 `NSApplication.currentEvent` 代替。应用内监听仅保留一次有效按下，松开或消费后释放；请求匹配原始按下点并绑定该次按下的代际和目标窗口，过期请求不能消费后来的点击。不安装全局监听，不吞掉事件，不自行求解拖动位置。
- 窗口与渲染时序按实际后端验证，包括 egui 多 pass 重绘和连续原生事件。图片绘制不能产生主窗口几何副作用，不能依赖某个可能被丢弃的 UI pass 提交 drawable 补丁。不得用巨大透明窗口、手动更新 GL context 或定位重试掩盖问题。
- 原生状态查询也要验证是否会改变窗口；macOS 贴图先在隐藏状态完成后端初始化，再显示，不能依赖隐藏窗口一定会绘制首帧。依赖优先使用官方发布包，不将第三方库源码整包复制到仓库，不修改本机依赖缓存或用构建脚本注入补丁；需要依赖修复时优先采用上游修复，不通过坐标补偿掩盖后端副作用。
- Windows 持续输入会饿死其他窗口的重绘，当前固定上游 egui PR #8650 的完整提交修复调度。所有 egui 类型保持同一来源，只允许规范中明确的 Git 仓库并要求固定 rev；不能跟随浮动分支。此开发依赖还包含图像 API 与字体依赖变更，更新时须检查三平台、网页、原图复制、许可证与体积。正式发布包包含该修复且通过原生持续输入回归后，撤销 Git 覆盖并恢复注册表依赖。
- 截图线程使用有界队列，取消立即恢复 UI、丢弃晚到结果并阻止尚未开始的后续屏幕采集，不承诺强停正在执行的系统调用。像素转换在工作线程完成，截图数据与纹理上传共享不可变图片；上传前验证 GPU 纹理边长上限。多屏并行不跨线程搬运非 Send 原生句柄，Linux Portal 不并发请求。
- GUI 沙箱复用实际 Rust 视图和几何，不复制一套 HTML 产品界面；仅使用生成图片，禁止真实截图、系统剪贴板、权限、自启和全局快捷键操作。多 agent 评审使用各自独立的 headless 浏览器上下文；浏览器交互和隐藏原生几何验证都不能替代合成器视觉验收。
- 原生版和网页演示共用交互规则，不只共用绘制：快捷键录制与校验统一在 `src/ui/shortcut.rs`，截图请求复用 `CaptureGate`，选区映射复用 `geometry::pin_placement`，关闭贴图统一调用 `ui::pin::forget` 清理列表、编辑和输入状态。Esc 只关闭持有键盘焦点的贴图；菜单和快捷键录制不得透传输入到背后的贴图，文本编辑按键不得操纵贴图。网页只模拟窗口、焦点和平台能力，不另写排序、快捷键或坐标规则。网页宿主的生成数据回归纳入工作区测试，覆盖完整输入到动作的路径；确需平台差异时在适配层说明原因，不为统一系统窗口行为引入补偿。

## Structure

- `src/capture.rs`, `src/capture/worker.rs`: monitor enumeration, shared in-memory pixels and bounded cancellable capture work.
- `src/capture/windows.rs`, `src/capture/windows/`: one-shot visible-window metadata, platform stacking/visibility adapters and source-pixel region mapping.
- `src/geometry.rs`: validated pixel selection and original-position pin placement.
- `src/pin.rs`: initial proportional fitting, display coordinate conversion and adaptive visible image controls placement.
- `src/app.rs`: capture state machine, events, and window lifecycle.
- `src/ui/`: launcher, selection overlay, floating image views, shared style profiles, and copy artwork.
- `preview/`: WebAssembly demo and local sandbox using the same Rust UI and fictional documents; no native services. Public builds use a redistributable font; local system fonts stay local.
- `src/app/tests.rs`, `src/ui/pin/tests.rs`, `src/ui/launcher/tests.rs`: lifecycle and view regressions, separate from production logic.
- `src/platform.rs`: keyboard shortcut and local system-font adapters.
- `src/platform/macos.rs`: explicit application activation and floating levels, preserving backend classes and observation lifecycle.
- `src/platform/macos_drag.rs`: retain the original press and bind deferred native drag requests to it, without computing movement.
- `src/platform/pin_window.rs`: same-process native pin lookup and explicit keyboard-focus activation without geometry changes.
- `src/platform/pin_zoom.rs`: atomic native anchored zoom using observed content bounds and per-pin native handles.
- `src/ui/pin/session.rs`: continuous input ownership and fixed zoom anchors, without window geometry prediction.
- `src/ui/pin/keyboard.rs`: system-repeat-driven arrow steps with bounded stride growth, without animation or geometry prediction.
- `src/platform/macos_dock.rs`, `packaging/macos/DockMenu.xib`: native Dock menu, system Quit and Reopen routing, independent of optional menu resource loading; Xcode compiles the NIB at bundle time.
- `src/platform/quit_prompt.rs`: asynchronous native quit confirmation with an application-UI fallback.
- `src/settings.rs`: shortcut validation, theme/icon preferences, and local storage.
- `src/i18n.rs`: typed Chinese/English catalogue, deferred messages and locale matching.
- `src/permission.rs`: user-driven authorization state without native calls or polling.
- `src/platform/macos_access.rs`: main-thread permission requests, settings navigation, and installation/identity migration checks.
- `src/tray.rs`: event-driven native menu bar/tray commands and status artwork.
- `src/startup.rs`: explicit current-user login registration with atomic, escaped file entries or Windows Run registry values.
- `assets/icons/`, `src/icon.rs`: vector sources, LFS PNG assets, and icon selection.
- `tests/`: deterministic checks using generated images.
- `packaging/`: platform packaging metadata and scripts.
- `.github/workflows/ci.yml`: format, Clippy, tests, and builds on all three OSes.
- `xtask/`: shared Rust checks, package/manifest validation, license notices and manual release commands.
- `docs/workflow.md`: contributor flow, native acceptance, approved CI signing and manual publication.
- `.github/rulesets/`: versioned main/tag protection definitions, applied and verified through GitHub.
