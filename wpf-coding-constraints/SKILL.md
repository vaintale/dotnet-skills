---
name: wpf-coding-constraints
description: 约束 WPF/XAML 代码实现规范，强制遵循微软官方 WPF 指南（MVVM、数据绑定、依赖属性、命令、线程模型、性能）。当用户在本项目新增或修改 WPF 代码时使用。不用于 Web API 或 WinForms。
---

# WPF 代码实现规范约束

本 skill 约束在本项目中编写 WPF（.NET / C# / XAML）代码时的实现规范，约束依据来自微软官方文档（Microsoft Learn，learn.microsoft.com）。凡涉及 WPF 代码的新增、修改、重构，都必须遵守以下规则。

## 总则

1. 一切约束以微软官方 WPF 文档为准。当规则冲突或不确定时，优先查阅下方「官方约束来源」中的文档，不得自行发挥。
2. 编写前先读取相关目标文件全文，并用 grep 定位所有调用方，控制改动半径，保持最小 diff。
3. 改动后必须能够通过 `dotnet build` 编译，并说明改动边界与影响。

## 官方约束来源（Microsoft Learn）

- WPF 文档首页：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/
- 数据绑定概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/data/data-binding-overview
- 依赖属性概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/properties/dependency-properties-overview
- 路由事件概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/routed-events-overview
- 命令概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/commanding-overview
- 输入概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/input-overview
- XAML 资源概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/systems/xaml-resources-overview
- 样式与模板概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/controls/styles-templates-overview
- 线程模型：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/threading-model
- 性能优化：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/advanced/optimizing-performance
- XAML 概述：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/xaml/
- 布局：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/controls/layout
- 应用程序生命周期：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/app-development/
- 窗口与对话框：https://learn.microsoft.com/en-us/dotnet/desktop/wpf/windows/ 与 https://learn.microsoft.com/en-us/dotnet/desktop/wpf/windows/dialog-boxes-overview

## 架构分层（MVVM）

来源：微软 .NET 架构文档「Model-View-ViewModel 模式」。

- View（XAML + 代码隐藏）只负责呈现与交互，不得包含业务逻辑。
- ViewModel 不得引用任何 WPF 控件类型（禁止 using System.Windows.Controls 等 UI 命名空间），只依赖数据与命令。
- Model 只承载数据与领域逻辑，不感知 UI。
- 通过 DataContext 关联 View 与 ViewModel；禁止在代码隐藏里手动 new 控件并拼接业务逻辑。

## 应用程序生命周期

来源：应用开发（learn.microsoft.com/en-us/dotnet/desktop/wpf/app-development/）。

- 用 Application 类管理应用生命周期：StartupUri 或 Startup 事件指定启动窗口，ShutdownMode 控制进程何时退出。
- 应用级逻辑（单实例、启动参数、全局资源、退出清理）放在 App.xaml.cs，不得散落到各窗口。

## 窗口与对话框

来源：WPF 窗口（windows/）与对话框（dialog-boxes-overview）。

- 主窗口通过 StartupUri 或 Startup 指定；运行期通过 Application.Current.MainWindow / Application.Current.Windows 访问。
- 模态对话框用 ShowDialog()，非模态用 Show()；模态窗口务必设置 Owner，保证置顶与关闭联动。
- 通用对话框（打开/保存文件、文件夹、颜色等）用 Microsoft.Win32 的 OpenFileDialog / SaveFileDialog / OpenFolderDialog，不自己造。
- 关闭语义明确：模态窗口用 DialogResult 返回结果，主窗口关闭用 Close()，并核对 ShutdownMode 避免窗口全关后进程残留。

## XAML 与代码隐藏

来源：XAML 概述（learn.microsoft.com/en-us/dotnet/desktop/wpf/xaml/）。

- UI 结构与布局优先用 XAML 声明式编写。
- 代码隐藏（code-behind）保持最小化，仅处理 XAML 无法表达的逻辑（窗口生命周期、拖放等）。
- 禁止把可写成绑定的逻辑硬编码进事件处理程序。

## 数据绑定

来源：数据绑定概述。

- 优先用数据绑定代替手动给控件属性赋值、手动监听事件同步数据。
- 正确设置 Binding Mode（OneWay / TwoWay / OneTime / OneWayToSource）；需要即时回写时设置 `UpdateSourceTrigger=PropertyChanged`。
- 绑定源必须实现 `INotifyPropertyChanged` 提供变更通知，否则 OneWay/TwoWay 不会自动刷新。
- 绑定集合时使用 `ObservableCollection<T>`，保证增删自动刷新界面。

## 数据转换

来源：数据绑定概述「Data conversion」章节。

- 源类型与目标类型不匹配时，用 IValueConverter 实现转换，禁止把类型转换逻辑写死在 ViewModel 或控件里。
- 转换器实现 Convert / ConvertBack，注册为资源后通过 StaticResource 引用。

## 数据校验

来源：数据绑定概述「Data validation」章节。

- 输入校验用 INotifyDataErrorInfo（.NET 现代推荐）或 ValidationRule，不用已过时的 IDataErrorInfo。
- 校验失败通过绑定反馈到界面（ValidatesOnNotifyDataErrors / Validation.ErrorTemplate），禁止静默吞掉校验错误。

## 属性变更通知

- ViewModel 属性在 setter 中触发 `PropertyChanged`，属性名用 `nameof` 或 `[CallerMemberName]`，禁止手写字符串魔法值。
- 仅在实际值发生变化后才触发通知，避免无意义刷新。

## 命令

来源：命令概述。

- 用户操作（按钮点击、菜单等）用 `ICommand` / `RelayCommand` 实现，禁止在代码隐藏写 Click 事件处理程序执行业务逻辑。
- 正确实现 `CanExecute`，并通过 `CanExecuteChanged` 通知可用性变化。

## 路由事件

来源：路由事件概述。

- 理解冒泡（Bubbling）与隧道（Tunneling / Preview）两种路由方向，按场景选择正确的处理阶段。
- 需要阻止事件继续路由时，正确设置 e.Handled = true。
- 自定义控件要对外提供事件时，优先复用现有路由事件或注册自定义 RoutedEvent（含 AddOwner），不用普通 CLR 事件。

## 输入

来源：输入概述。

- 输入处理优先用命令（ICommand）或数据绑定，禁止在代码隐藏写 KeyDown / MouseDown / TextInput 处理业务逻辑。
- 焦点与 Tab 顺序用 FocusManager / Focusable 管理，遵循可预期的键盘导航。
- 需要捕获鼠标时用 CaptureMouse，并在结束后 ReleaseMouseCapture。

## 依赖属性

来源：依赖属性概述。

- 仅当属性需要参与绑定、样式、动画、继承、默认值等 WPF 属性系统特性时，才定义为 `DependencyProperty`。
- 否则使用普通 CLR 属性，不得为「看起来像 WPF」而滥用依赖属性。
- 自定义依赖属性必须遵循注册约定：`public static readonly DependencyProperty` 字段，CLR 包装属性只调用 `GetValue` / `SetValue`。

## 资源

来源：XAML 资源概述。

- 复用对象（画刷、样式等）用资源（ResourceDictionary）定义。
- 静态不变的资源用 `StaticResource`，运行期可能变化的资源用 `DynamicResource`，正确区分二者。

## 样式与模板

来源：样式与模板概述。

- 复用外观用 `Style`，定制控件结构用 `ControlTemplate`，数据呈现用 `DataTemplate`，正确选择。
- 避免为每个控件重复内联样式，提取到资源或主题。

## 线程模型

来源：线程模型。

- 所有 UI 元素只能由创建它们的 UI 线程（Dispatcher 线程）访问。
- 后台线程禁止直接访问 UI 元素；跨线程更新用 `Dispatcher.Invoke` / `Dispatcher.BeginInvoke`，或用 async/await 让延续回到 UI 上下文。
- 阻塞的耗时操作用 `Task.Run` 放入后台线程，避免阻塞 UI。

## 异步

- async 方法中禁止用 `.Wait()` / `.Result` 阻塞 UI 线程。
- 仅事件处理程序允许 `async void`，其余一律返回 `Task`。
- 后台延续若不需要 UI 上下文，使用 `ConfigureAwait(false)`。

## 性能

来源：性能优化。

- 大型列表必须启用虚拟化（`VirtualizingStackPanel` / `VirtualizingPanel`）。
- 可跨线程共享的 `Freezable` 对象（如 `Brush`）应调用 `Freeze()`。
- 避免不必要的布局重算与过度嵌套面板。

## 命名与组织

- 遵循 .NET 命名规范：类型/属性/方法用 PascalCase，局部变量/字段用 camelCase。
- XAML 元素仅在需要引用时才用 `x:Name` 命名，且语义化，禁止 x:Name 泛滥。

## 低频主题

以下主题微软官网同样有专门文档，但多数业务型 WPF 项目较少用到。遇到相关改动时，加载 [references/low-frequency-topics.md](references/low-frequency-topics.md) 中的对应约束：

- 图形与多媒体（2D/3D/动画/媒体）
- 全球化与本地化
- 安全
- 文档与打印
- 互操作（Win32 / Windows Forms / Direct3D9）
- 部署（ClickOnce / MSIX）
- 异常处理
- 可访问性（UI Automation）

## 交付说明（改动后必给）

改动 WPF 代码后，必须给出四段说明：

1. 改了什么（文件与位置）。
2. 为什么这样改（对应哪条微软官方约束）。
3. 影响范围（哪些 View / ViewModel / Model 受影响）。
4. 验证方式（`dotnet build` 结果、绑定是否成立）。
