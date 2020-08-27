---
title: 1px 细线实现
date: 2020-08-26 20:26:13
category: CSS3
tags: CSS3
---

采用缩放方案，单条边线好处理一点；如果需要四边都有边线，会有点麻烦，使用缩放方案，虽然可行，但是在部分设备上有间隙问题，建议放弃，或者使用 svg 来做 background-image。

以下是 less/sass 的简单实现示例。

### LESS 上边线 1px

```less
/**
 * 1px细线 - 上边线
 */
.border-top(@color) {
  position: relative;

  &:before {
    position: absolute;
    top: 0;
    left: 0;
    content: '\0020';
    width: 100%;
    height: 1px;
    border-top: 1px solid @color;
    transform-origin: 0 0;
    overflow: hidden;
  }

  @media (-webkit-min-device-pixel-ratio: 1.5) and (-webkit-max-device-pixel-ratio: 2.49) {
    &:before {
      transform: scaleY(0.5);
    }
  }

  @media (-webkit-min-device-pixel-ratio: 2.5) {
    &:before {
      transform: scaleY(0.33333);
    }
  }
}

// 使用
.test {
  .border-top(#eee);
}
```

### SASS 上下 1px 细线

```scss
/**
 * 1px细线
 *  $position top/bottom
 */
@mixin border-1px($position, $color) {
  position: relative;

  &:before {
    position: absolute;
    @if $position == top {
      top: 0;
    }
    @if $position == bottom {
      bottom: 0;
    }
    left: 0;
    content: '\0020';
    width: 100%;
    height: 1px;
    border-#{$position}: 1px solid $color;
    transform-origin: 0 0;
    overflow: hidden;
  }

  @media (-webkit-min-device-pixel-ratio: 1.5) and (-webkit-max-device-pixel-ratio: 2.49) {
    &:before {
      transform: scaleY(0.5);
    }
  }

  @media (-webkit-min-device-pixel-ratio: 2.5) {
    &:before {
      transform: scaleY(0.33333);
    }
  }
}

// 使用
.test {
  // 上边线
  @include border-1px(top, #e9e9e9);
  // 下边线
  @include border-1px(bottom, #e9e9e9);
}
```
