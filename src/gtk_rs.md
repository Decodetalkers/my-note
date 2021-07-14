# GTK-rs

我之前有过使用gtk-rs，但是在一次事故中东西都没有了。我一直认为这个库特别屑，但是重新看了并不是这么一回事情。

接下来我想到哪里写道哪里

## 基本组件

GTK-rs和gtk本身的组件是一致的，

* gtk::Application
    * Gtk 的事件，用户绑定各种活动
* gtk::ApplicationWindow
    * 定义了一个 活动窗口需要，是核心
* gtk::Box
    * 布局
* gio::Menu
    * 菜单，需要通过application加上。application里有俩菜单，一个是*set_app_menu*,定义了"应用菜单"，另一个是set_menu_bar,这是个菜单，没有名字。

## 事件

gtk-rust的事件相当有趣。其中一个重要的库是gui。这个和我之前用的tui库非常想，通过名字查找组件可以获取注册的组件绑定，这个需要通过注册中以特定语法注册才会访问比如menu

```rust
menu.append(Some("Quit"),Some('app.quit'));
```

这样定义后就可以通过app.后面的字段访问

```rust
let quit = gio::SimpleAction::new("quit",None);
```
