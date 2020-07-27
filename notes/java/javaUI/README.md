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

