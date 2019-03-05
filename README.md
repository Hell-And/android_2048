# android_2048
Android原生，小游戏2048

# 效果
![](https://github.com/LoverAnd/android_2048/blob/master/android2048.gif)

# 核心代码


## 手势滑动
``` java
private void left() {
        //左右都是 遍历行
        for (int i = 0; i < rowsCount; i++) {
            clone = datas.get(i).clone();
            //数据左移
            clonecache = new int[clone.length];
            datas.remove(i);
            int left = 0;//左侧开始计算
            int right = clone.length - 1;//右侧开始计算
            int data = 0;
            //将[0,2,0,0] or [0,2,2,0] 这种 先转换为 [2,0,0,0] or [4,0,0,0] 即 数据左移
            for (int j = 0, jj = clone.length; j < jj; j++) {
                if (clone[j] == 0) {
                    clonecache[right--] = 0;
                } else {
                    BlockCoordinateXY blockCoordinateXY = null;
                    if (data == clone[j]) {//如果这个值 与 前面保存的 不为0的值相同 就会相加 (碰撞相加)
                        //添加会运动的格子
                        blockCoordinateXY = new BlockCoordinateXY((BlockView) getChildAt(i * clone.length + j), (i * clone.length + j), (i * clone.length + left - 1));
                        startblockCoordinateXYList.add(blockCoordinateXY);
                        clonecache[left - 1] = data + clone[j];
                        //计算分数
                        addMultiple(data + clone[j]);
                        data = 0;
                        //添加产生加法的格子
                        blockindex.add(i * clone.length + left - 1);
                    } else {
                        //添加会运动的格子
                        blockCoordinateXY = new BlockCoordinateXY((BlockView) getChildAt(i * clone.length + j), (i * clone.length + j), (i * clone.length + left));
                        startblockCoordinateXYList.add(blockCoordinateXY);
                        //保存这一个不为0的值
                        data = clone[j];
                        clonecache[left++] = clone[j];
                    }
                }
            }
            datas.add(i, clonecache);
        }
        //准备动画
        preAnimation("translationX");
    }
```
## 动画
```java
 private void preAnimation(String direction) {
        if (objectAnimatorList == null) {
            objectAnimatorList = new ArrayList<>();
        } else {
            objectAnimatorList.clear();
        }

        for (int i = 0; i < startblockCoordinateXYList.size(); i++) {
            ObjectAnimator objectAnimator = null;
            BlockCoordinateXY blockCoordinateXY = startblockCoordinateXYList.get(i);
            if (blockCoordinateXY != null) {
                //判断动画方向 distance是动画距离
                if (direction.equals("translationX")) {
                    distance = pointFS.get(blockCoordinateXY.getTo()).x + blockSideLength
                            - (pointFS.get(blockCoordinateXY.getFrom()).x + blockSideLength);

                } else {
                    distance = pointFS.get(blockCoordinateXY.getTo()).y + blockSideLength
                            - (pointFS.get(blockCoordinateXY.getFrom()).y + blockSideLength);

                }
                objectAnimator = ObjectAnimator.ofFloat(blockCoordinateXY.getBlockView(), direction, distance);
                objectAnimatorList.add(objectAnimator);
            }
        }

        if (objectAnimatorList.size() > 0) {
            //开始动画
            starAnimate();
        }
    }
```
## 绘制
```java
        //先移除子view 再添加
        removeAllViews();
        for (int i = 0, ii = datas.size(); i < ii; i++) {
            final int[] intarr = datas.get(i);
            for (int j = 0, jj = intarr.length; j < jj; j++) {
                BlockView textView = new BlockView(getContext());
                params = new GridLayout.LayoutParams();
                textView.setText(intarr[j]);
                // 设置行列下标，和比重
                params.rowSpec = GridLayout.spec(i, 1, 1f);
                params.columnSpec = GridLayout.spec(j, 1, 1f);
                params.setMargins(space, space, space, space);
                //这两句要加 不加宽度会出问题
                params.width = 0;
                params.height = 0;
                textView.setLayoutParams(params);
                addView(textView, params);

            }
        }
```
其它的颜色，撤回，新游戏随机格子就简单了
