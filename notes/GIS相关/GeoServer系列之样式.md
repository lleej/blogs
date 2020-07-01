title: GeoServer系列之样式
date: 2020-06-04
tags: GIS
categories: GIS
layout: traft

------

摘要：介绍`GeoServer`中样式的配置和应用细节...

<!-- more -->

## 前言

地理空间数据没有内在的视觉成分。为了让用户更舒适的查看数据，必须对其进行样式设置（**设置样式是一件很重要的复杂事情，不次于数据采集、加工的重要性）

样式指定用于在地图上呈现数据的颜色、厚度和其他可见属性。`GeoServer`可以提供：

- 矢量样式

  - 点、线、多边形
  - 笔触、填充
  - 颜色、线型、尺寸、透明度
- 栅格样式
  - 栅格数据
  - 调色板、透明度、对比度、其他参数

## SLD

样式化图层描述符（`Style Layer Descriptor`）是`XML`格式的文件

这是一个`SLD`样式文件的示例，描述的是一个点的样式（直径为`6`像素的红色圆圈）

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<StyledLayerDescriptor version="1.0.0"
    xsi:schemaLocation="http://www.opengis.net/sld StyledLayerDescriptor.xsd"
    xmlns="http://www.opengis.net/sld"
    xmlns:ogc="http://www.opengis.net/ogc"
    xmlns:xlink="http://www.w3.org/1999/xlink"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <NamedLayer>
    <Name>Simple point</Name>
    <UserStyle>
      <Title>GeoServer SLD Cook Book: Simple point</Title>
      <FeatureTypeStyle>
        <Rule>
          <PointSymbolizer>
            <Graphic>
              <Mark>
                <WellKnownName>circle</WellKnownName>
                <Fill>
                  <CssParameter name="fill">#FF0000</CssParameter>
                </Fill>
              </Mark>
              <Size>6</Size>
            </Graphic>
          </PointSymbolizer>
        </Rule>
      </FeatureTypeStyle>
    </UserStyle>
  </NamedLayer>
</StyledLayerDescriptor>
```

文件内容说明：

- 14行：规定为一个点样式`PointSymbolizer`
- 17行：设置为圆形（`circle`），默认支持：正方形（`square`）、星型（`star`）、三角形（`triangle`）等
- 19行：设置形状填充的颜色，使用`RRGGBB`样式的颜色值
- 22行：设置形状的尺寸

#### 符号编码样式

`Symbology Encoding Style`，是指`SLD`文件中只包含一个`NamedLayer`元素，且该元素中只包含一个`UserStyle`元素

#### 样式层描述符样式

`Style Layer Descriptor Styles`，是指`SLD`文件中包含任意数量的`<NamedLayer>`和`<UserLayer>`元素，每个元素可以包含任意数量的`<UserStyle>`或`<NamedStyle>`元素

没有搞清楚这两个的区别？？？？？？？

### 点样式（Point）

点样式：针对地图数据中类型为`Point`的矢量数据而设置的样式，分为简单的单一规则样式和复杂的多规则样式

#### 单一规则样式

在样式文件中，只有一个`Rule`元素，单一样式中，可以配置的样式项目有：

- `PointSymbolizer`：点样式
  - `Graphic`：图形样式
    - `Mark`：简单形状
      - `WellKnownName`：形状名（圆、方、三角、星星等）
      - `Fill`：填充样式
        - `CssParameter`：样式参数
          - `name="fill"`：填充颜色（`#FF0000`红色）
          - `name="fill-opacity"`：填充透明度（`0.2`）
      - `Stroke`：笔触样式
        - `CssParameter`：样式参数
          - `name="stroke"`：笔触颜色（`#000000`黑色）
          - `name="stroke-width"`：笔触宽度（`2`）
    - `ExternalGraphic`：使用图形
      - `OnlineResource`：图形资源
        - `xlink:type="simple"`：简单类型
        - `xlink:href="smileyface.png"`：图形资源地址        
      - `Format`：图形格式（`image/png`）
    - `Size`：形状尺寸（`2`）
      - 如果是`ExternalGraphic`，且图形是矩形，则控制其高度，而宽度按比例缩放
    - `Rotation`：形状旋转角度，顺时针（`-360 - 360`）
- `TextSymbolizer`：文本样式
  - `Label`：标签
    - `ogc:PropertyName`：标签显示数据中属性的值（`name`：点的名称）
  - `Fill`：文本样式
    - `CssParameter`：样式参数
      - `name="fill"`：文本颜色（`#000000`黑色）
  - `Font`：字体样式
    - `CssParameter`：样式参数
      - `name="font-family"`：字体系列（`Arial`）
      - `name="font-size"`：字体大小（`12`）
      - `name="font-style"`：字体样式（`normal`）
      - `name="font-weight"`：字体粗细（`bold`）
  - `LabelPlacement`：标签位置
    - `PointPlacement`：相对于点位置
      - `AnchorPoint`：对齐的锚点
        - `AnchorPointX`：水平方向（0-左、0.5-居中、1-右）
        - `AnchorPointY`：垂直方向（0-底、0.5-居中、1-上）
      - `Displacement`：标签与点的偏移量
        - `DisplacementX`：水平偏移像素（`0`）
        - `DisplacementY`：垂直偏移像素（`5`）
      - `Rotation`：标签旋转角度，顺时针（`-360 - 360`）

```xml
<FeatureTypeStyle>
     <Rule>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#FF0000</CssParameter>
             </Fill>
           </Mark>
           <Size>6</Size>
         </Graphic>
       </PointSymbolizer>
       <TextSymbolizer>
         <Label>
           <ogc:PropertyName>name</ogc:PropertyName>
         </Label>
         <Font>
           <CssParameter name="font-family">Arial</CssParameter>
           <CssParameter name="font-size">12</CssParameter>
           <CssParameter name="font-style">normal</CssParameter>
           <CssParameter name="font-weight">bold</CssParameter>
         </Font>
         <LabelPlacement>
           <PointPlacement>
             <AnchorPoint>
               <AnchorPointX>0.5</AnchorPointX>
               <AnchorPointY>0.0</AnchorPointY>
             </AnchorPoint>
             <Displacement>
               <DisplacementX>0</DisplacementX>
               <DisplacementY>25</DisplacementY>
             </Displacement>
             <Rotation>-45</Rotation>
           </PointPlacement>
         </LabelPlacement>
         <Fill>
           <CssParameter name="fill">#990099</CssParameter>
         </Fill>
       </TextSymbolizer>
     </Rule>
   </FeatureTypeStyle>
```

#### 多规则样式

在一个点的样式文件中，包含多个`Rule`元素，一般情况下，都是针对`Point`的属性值而区分不同的样式规则

多规则样式是在单一规则样式中，增加了一些特殊的元素，这些元素在每个`Rule`元素中

##### 基于属性值的显示

可以根据属性值的不同，设置不同的样式规则，让数据呈现出不同的效果

- `Name`：规则的名称（用于区分，如：`SmallPop`）
- `Title`：规则的标题（简要说明，如：`1 to 50000`）
- `ogc:Filter`：规则的过滤条件
  - `ogc:And`：关系运算符（`not`、`and`、`or`）
    - `ogc:PropertyIsLessThan`：比较运算符：小于
      - `ogc:PropertyName`：属性名称（如：`pop`）
      - `ogc:Literal`：条件中的数值（如：`50000`）
    - `ogc:PropertyIsGreaterThanOrEqualTo`：比较运算符-大于等于

```xml
<FeatureTypeStyle>
     <Rule>
       <Name>SmallPop</Name>
       <Title>1 to 50000</Title>
       <ogc:Filter>
         <ogc:PropertyIsLessThan>
           <ogc:PropertyName>pop</ogc:PropertyName>
           <ogc:Literal>50000</ogc:Literal>
         </ogc:PropertyIsLessThan>
       </ogc:Filter>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#0033CC</CssParameter>
             </Fill>
           </Mark>
           <Size>8</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
     <Rule>
       <Name>MediumPop</Name>
       <Title>50000 to 100000</Title>
       <ogc:Filter>
         <ogc:And>
           <ogc:PropertyIsGreaterThanOrEqualTo>
             <ogc:PropertyName>pop</ogc:PropertyName>
             <ogc:Literal>50000</ogc:Literal>
           </ogc:PropertyIsGreaterThanOrEqualTo>
           <ogc:PropertyIsLessThan>
             <ogc:PropertyName>pop</ogc:PropertyName>
             <ogc:Literal>100000</ogc:Literal>
           </ogc:PropertyIsLessThan>
         </ogc:And>
       </ogc:Filter>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#0033CC</CssParameter>
             </Fill>
           </Mark>
           <Size>12</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
     <Rule>
       <Name>LargePop</Name>
       <Title>Greater than 100000</Title>
       <ogc:Filter>
         <ogc:PropertyIsGreaterThanOrEqualTo>
           <ogc:PropertyName>pop</ogc:PropertyName>
           <ogc:Literal>100000</ogc:Literal>
         </ogc:PropertyIsGreaterThanOrEqualTo>
       </ogc:Filter>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#0033CC</CssParameter>
             </Fill>
           </Mark>
           <Size>16</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
   </FeatureTypeStyle>
```

##### 基于属性值的缩放

可以根据属性值的不同，在不同缩放比例下呈现出不同的效果

- `MaxScaleDenominator`：最大缩放比例值
- `MinScaleDenominator`：最小缩放比例值（如：`10000`）

缩放比例：`10000`表示地图投影单位比例为`1: 10000`

```
<Name>Large</Name>
       <MaxScaleDenominator>160000000</MaxScaleDenominator>
```

```xml
<FeatureTypeStyle>
     <Rule>
       <Name>Large</Name>
       <MaxScaleDenominator>160000000</MaxScaleDenominator>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#CC3300</CssParameter>
             </Fill>
           </Mark>
           <Size>12</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
     <Rule>
       <Name>Medium</Name>
       <MinScaleDenominator>160000000</MinScaleDenominator>
       <MaxScaleDenominator>320000000</MaxScaleDenominator>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#CC3300</CssParameter>
             </Fill>
           </Mark>
           <Size>8</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
     <Rule>
       <Name>Small</Name>
       <MinScaleDenominator>320000000</MinScaleDenominator>
       <PointSymbolizer>
         <Graphic>
           <Mark>
             <WellKnownName>circle</WellKnownName>
             <Fill>
               <CssParameter name="fill">#CC3300</CssParameter>
             </Fill>
           </Mark>
           <Size>4</Size>
         </Graphic>
       </PointSymbolizer>
     </Rule>
   </FeatureTypeStyle>
```

### 线样式（Line）

线样式：针对地图数据中类型为`Line`的矢量数据而设置的样式

**线样式中：没有`Fill`元素**

线样式只能使用`stroke`笔触，而不能使用`fill`填充。另外，对于标签的控制还有一些特殊的元素

- `LineSymbolizer`：线样式
  - `Stroke`：笔触样式
    - `CssParameter`：样式参数
      - `name="stroke"`：线颜色
      - `name="stroke-width"`：线宽度（像素）
      - `name="stroke-dasharray"`：虚线，值：实线长度 空白长度，如：`5 2`
      - `name="stroke-dashoffset"`：虚线偏移量，至少有两条**虚线样式**
      - `name="stroke-linecap"`：边缘形状，值：`round`为圆弧
    - `GraphicStroke`：用图形画线段
      - `Graphic`：图形
        - `Mark`：使用符号
          - `WellKnownName`：常用名
          - `Stroke`：笔触...
        - `Size`：大小
  - `PerpendicularOffset`：线偏移（像素），至少有两条**线样式**
- `TextSymbolizer`：文本样式
  - `Label`：标签...
  - `Fill`：文本样式...
  - `Font`：字体样式...
  - `LabelPlacement`：标签位置
    - `LinePlacement`：相对于线的位置
  - `VendorOption`：
    - `name="followLine"`：是否跟随线型（`true`、`false`）
    - `name="maxAngleDelta"`：标签将跟随的最大角度（如：`90`）
    - `name="maxDisplacement"`：标签的最大位移（像素）（如：`400`）
    - `name="repeat"`：重叠时，尝试移动的位移（像素）（如：`150`）

要想实现带边框的线段，需要连续画两条线，第一条是底线（即边框），第二条是中线（即填充）

**注意：必须先画底线（宽），再画中线（窄），否则底线会覆盖中线**

```xml
<FeatureTypeStyle>
      <Rule>
       <LineSymbolizer>
         <Stroke>
           <CssParameter name="stroke">#333333</CssParameter>
           <CssParameter name="stroke-width">5</CssParameter>
           <CssParameter name="stroke-linecap">round</CssParameter>
         </Stroke>
       </LineSymbolizer>
       <LineSymbolizer>
       <Stroke>
           <CssParameter name="stroke">#6699FF</CssParameter>
           <CssParameter name="stroke-width">3</CssParameter>
           <CssParameter name="stroke-linecap">round</CssParameter>
         </Stroke>
       </LineSymbolizer>
      </Rule>
   </FeatureTypeStyle>
```

`GeoServer`会确保不会将标签绘制在其他标签之上，从而使两者都变得模糊

### 多边形（Polygons）

多边形：是由首位相接的线组成的区域，样式包含了笔触（`stroke`）和填充（`fill`），与点样式类似

- `PolygonSymbolizer`：多边形样式
  - `Stroke`：笔触...
  - `Fill`：填充...
- `LineSymbolizer`：内部缓冲线
  - `PerpendicularOffset`：偏移量（像素）
- `TextSymbolizer`：文本样式
  - `Label`：标签...
  - `Fill`：文本样式...
  - `Font`：字体样式...
  - `LabelPlacement`：标签位置
    - `PointPlacement`：相对于**多边形质心**位置
      - `AnchorPoint`：对齐的锚点
        - `AnchorPointX`：水平方向（0-左、0.5-居中、1-右）
        - `AnchorPointY`：垂直方向（0-底、0.5-居中、1-上）
  - `Halo`：光晕样式
    - `Radius`：晕轮半径（像素）
    - `Fill`：晕轮填充...
  - `VendorOption`：
    - `name="autoWrap"`：标签一行宽度（像素），超过宽度自动换行
    - `name="maxDisplacement"`：标签的最大位移（像素）（如：`400`）

### 栅格（Raster）

栅格是显示在网格中的地理数据。与图像文件（如`PNG`文件）相似，不同之处在于，每个点都包含数字形式的地理信息，而不是每个点都包含视觉信息。栅格可以看作是数值的地理参考表

栅格的一个示例是数字高程模型（`DEM`）图层，该图层的高程数据在每个地理参考数据点处进行了数字编码

- `RasterSymbolizer`：栅格样式
  - `ColorMap`：颜色范围（参数`type="intervals"`代表**离散**而不是**平滑渐变**）
    - `ColorMapEntry color="#008000" quantity="70"`：下限颜色项（本例为`70`最下限）**必须**
    - ...
    - `ColorMapEntry color="#BBBB00" quantity="250" opacity="0"`：上限颜色项（本例为`250`最上限）**必须**
  - `Opacity`：透明度（`0 - 1`）
  - `ContrastEnhancement`：对比度
    - `Normalize`：对比度标准化输出
    - `GammaValue`：亮度值（`0-1`），`0`最亮
    - 