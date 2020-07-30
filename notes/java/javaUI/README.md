# Java-UI

## 获取屏幕尺寸

```java
Dimension dimension = Toolkit.getDefaultToolkit().getScreenSize();
jframe.setSize(dimension.width, dimension.height);
```

## 设置最大化

```java
jFrame.setExtendedState(JFrame.MAXIMIZED_BOTH);
```

## 是否显示标题栏

```java
jFrame.setUndecorated(true)
```

## 设置按钮背景色

```java
jButton.setOpaque(true);
jButton.setBackground(Color.white);
```

