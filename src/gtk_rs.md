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

### 匿名函数与glib::clone!

众所周知，rust的匿名函数对于外部变量一般只有访问的权限，无权更改，但是对于gtk-rs里的东西，有它的trait，就能使用

```rust
glib::clone!(@weak window => move |_|());
```

### 大量的引用

基本都是引用类型

### 资源管理

qt有个qrc，qt也有类似的管理工具。

资源管理如下

* Cargo.toml
* target
* src
    * main.rs
    * resources
        * a.png
        * b.png
        * resources.gresource.xml

resources.gresource.xml 里是这种管理模式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<gresources>
    <gresource prefix="/resources">
        <file>a.png</file>
        <file>b.png</file>
    </gresource>
</gresources>
```

这样定义后就可以通过pixmap引入icon编译了
```rust
use gtk::gdk_pixbuf::Pixbuf;
let img = Pixbuf::from_resouce("/resources/a.png");
```

同时有另一种不定义资源的方式实时获取指定目录下的icon

```rust
use gtk::gdk_pixbuf::Pixbuf
if let Ok(icon) = &Pixbuf::from_file("./youxie.jpeg") {
    //这个window是ApplicationWindow, set_icon是设置dock栏图标
    window.set_icon(Some(icon));
}
```

### 设置expand

不同的组件有不同的方法设置expand

#### overlay

Overlay是一个漂浮组件，可以漂浮于widget上方。

任何组件都可以用box设置

方法一是居中它设置需要访问组件的地址的引用，以overlay为例子

```rust
let overlay = gtk::Overlay::new;
let overlay_text = gtk::Label::new(Some("0"));
overlay_text.setwidget_name("overlay_label");
overlay_text.set_halign(gtk::Align::End);
overlay_text.set_valign(gtk::Align::Start);
let hbox = gtk::Box::new(gtk::Orientation::Horizontal, 0);
let but1 = gtk::Button::with_label("Click me!");
let but2 = gtk::Button::with_label("Or me!");
let but3 = gtk::Button::with_label("Why not me?");
hbox.add(&but1);
hbox.add(&but2);
hbox.add(&but3);
hbox.set_child_expand(&but1,true);
hbox.set_child_expand(&but2,true);
hbox.set_child_expand(&but3,true);
```

另一种是在add时候就设置expand和fill

```rust
let v_box = gtk::Box::new(gtk::Orientation::Vertical, 10);
let label = gtk::Label::new(Some("Nothing happened yet"));
let switch = gtk::Switch::new();
// a is widget, b is expand c is fill,d is padding
v_box.pack_start(&label, true, true, 0);
v_box.pack_start(&switch, true, true, 0);
```

如你所见，add时候可以用pack_start来设置更多参数

#### 函数式的表达方法

见 dialog_async.rs

之前tui有遇到函数式的表达方法，也许叫传递链更加标准？这种方法可以很爽的构建一个组件，比如

```rust
let button = gtk::ButtonBuilder::new()
    .label("Open Dialog")
    .halign(gtk::Align::Center)
    .valign(gtk::Align::Center)
    .visible(true)
    .build();
```

但是布局的话有遇到很多奇怪的问题，暂时不是很推荐这样布局？大概？

```rust
let window = gtk::ApplicationWindowBuilder::new()
    .application(application)
    .title("Dialog Async")
    .default_width(350)
    .default_height(70)
    .child(&button)
    .visible(true)
    .build();
```

加单个是没啥问题的。所有组件都有builder的函数，可以快速构建一个组件

#### Trait

众所周知，rust中trait是很重要的东西，没有trait，函数复用，设计，泛型都会遇到极大问题。gtk里面有个相当优秀的函数应对泛型（大概

比如

```rust

async fn dialog<W: IsA<gtk::Window>>(window: W){
}
```
这样所有和window一样的或者变异的，都可以被输入

#### Async

详细信息可以建 dialog_async.rs

异步，

```
button.connect_clicked(glib::clone!(@weak window => move |_| {
    glib::MainContext::default().spawn_local(dialog(window));
}));
```

启动dialog(window)是个异步函数，可以被绑定到button上，在button没有结束之前不会阻塞，但是button不会响应任何需求;

准确来说是整个windows都被禁止相应新事件了，但是window的拖动更改大小不受影响。
