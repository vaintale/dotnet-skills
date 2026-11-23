# WPF 低频主题约束（按需加载）

以下主题微软官网均有专门文档，但多数业务型 WPF 项目较少用到。仅在涉及相关代码改动时按需查阅本节。

## 图形与多媒体

来源：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/graphics-multimedia/

- 2D 矢量图形用 Shape / DrawingVisual / DrawingContext；静态内容优先 Drawings，避免逐帧重绘。
- 3D 用 Viewport3D / Model3D，仅在确需 3D 时引入，不为简单效果引入 3D 复杂度。
- 动画用 Storyboard + DoubleAnimation 等，遵循依赖属性与 Freezable 规则；可冻结的动画对象应 Freeze()。
- 音视频用 MediaElement，注意异常处理与资源释放。

## 全球化与本地化

来源：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/globalization-and-localization

- 可本地化的字符串禁止硬编码在 XAML 或代码里，用资源（Resource / Resx）承载。
- 用 x:Uid 标记可本地化元素；支持 RTL 语言时正确设置 FlowDirection。

## 安全

来源：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/security-wpf

- 禁止加载不可信的 XAML / BAML / XPS 内容（如 XamlReader.Load 加载外部输入）。
- WebBrowser 控件遵守功能控制与安全设置，避免在部分信任环境依赖不受支持的部署方式。

## 文档与打印

来源：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/documents 与 https://learn.microsoft.com/en-us/dotnet/desktop/wpf/documents/printing-overview

- 流式文档用 FlowDocument / FlowDocumentScrollViewer，固定版式用 FixedDocument / DocumentViewer。
- 打印用 PrintDialog + DocumentPaginator，遵循打印与打印系统管理。

## 互操作

来源：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/migration-and-interoperability

- 嵌入 WinForms 控件用 WindowsFormsHost，WPF 控件嵌入 WinForms 用 ElementHost。
- 与 Win32 交互用 HwndSource / HwndHost，注意 HWND 生命周期与消息循环。

## 部署

来源：https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/publish-first-app

- ClickOnce 支持 WPF / WinForms 自动更新；需要包标识或上架时用 MSIX。
- 独立 WPF 应用不支持部分信任部署。

## 异常处理

来源：应用开发（https://learn.microsoft.com/en-us/dotnet/desktop/wpf/app-development/）与 Application 类。

- 全局未处理异常用 Application.DispatcherUnhandledException 兜底，记录日志并决定是否退出。
- 避免应用静默崩溃，UI 线程异常要有统一处理入口。

## 可访问性

来源：https://learn.microsoft.com/en-us/dotnet/framework/ui-automation/ui-automation-overview

- WPF 通过 UI Automation 提供辅助功能（无 Active Accessibility 原生支持）。
- 自定义控件实现 AutomationPeer，暴露 AutomationControlType、名称、值等自动化属性。
