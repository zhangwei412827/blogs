+++
title = "贪吃蛇小游戏"
tags = ["javascript"]
date = "2013-03-25T9:51:02+08:00"
+++
### 贪吃蛇小游戏
```js
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>贪吃蛇 作者:记忆中的马肠河</title>
    <script type="text/javascript">

        window.onload = function () { 
            var snake = {
                //移动方向
                fangxiang: "left",
                //蛇身
                snakeBody: [],
                box: document.getElementById("snakeBox"),
                //移动
                move: function () {
                    var timer = "";
                    //判断是否GameOver
                    var isFailed = function () {
                        //判断蛇头是否碰到蛇身
                        var isHeadRunIntoBody=function(){
                            var t=false;
                            for(var i=0;i<snake.snakeBody.length-1;i++){
                                if(snake.snakeBody[snake.snakeBody.length-1].x==snake.snakeBody[i].x&&
                                snake.snakeBody[snake.snakeBody.length-1].y==snake.snakeBody[i].y
                                ){
                                t=true;
                                break;
                                }
                            }
                            return t;
                        }
                        //判断条件：是否蛇身碰到蛇头、是否蛇头碰到墙壁
                        if (
                        isHeadRunIntoBody()||
                        snake.snakeBody[snake.snakeBody.length - 1].x > snake.box.offsetLeft + parseInt(snake.box.style.borderWidth) + snake.box.clientWidth - 10
                        || snake.snakeBody[snake.snakeBody.length - 1].x < snake.box.offsetLeft + parseInt(snake.box.style.borderWidth)
                        || snake.snakeBody[snake.snakeBody.length - 1].y < snake.box.offsetTop + parseInt(snake.box.style.borderWidth)
                        || snake.snakeBody[snake.snakeBody.length - 1].y > snake.box.offsetTop + parseInt(snake.box.style.borderWidth) + snake.box.clientWidth - 10
                        ) {
                            //清除定时器
                            clearTimeout(timer);
                            alert("Game over!");
                            return false;
                        }
                    }
                    //存放蛇头
                    var snakeHead = snake.snakeBody[snake.snakeBody.length - 1];
                    //去除蛇尾 
                    var sheWei = snake.snakeBody.shift();
                    switch (snake.fangxiang) {
                        case "up":
                            sheWei.x = snakeHead.x;
                            sheWei.y = snakeHead.y - 10;
                            break;
                        case "down":
                            sheWei.x = snakeHead.x;
                            sheWei.y = snakeHead.y + 10;
                            break;
                        case "left":
                            sheWei.x = snakeHead.x - 10;
                            sheWei.y = snakeHead.y;
                            break;
                        case "right":
                            sheWei.x = snakeHead.x + 10;
                            sheWei.y = snakeHead.y;
                            break;
                        default:
                            break;
                    }
                    //把蛇尾放到蛇头
                    snake.snakeBody.push(sheWei);

                    //每次移动蛇身，都去吃食物.
                    snake.eatFood();
                    //是否失败
                    var ISF = isFailed();
                    //如果失败，不再绘制；
                    if (!ISF && ISF != undefined) {
                        return;
                    } else {
                        snake.drawSnake();
                    }
                    //蛇身移动的定时器
                    timer = setTimeout(snake.move, 500);
                },
                drawSnake: function () {
                    //清空已绘制的蛇身 
                     var bc = snake.box.childNodes;
                     for (var i = 0, len = bc.length; i < len; i++) {
                         var tn = bc[i];
                         if (tn) {
                          var tng = tn.tagName;
                           if (tng) {
                             var id = bc[i].getAttribute("id")
                             if (tng.toLowerCase() == "div" && id != "foodDiv") {
                              snake.box.removeChild(bc[i]);
                              //没删除一个节点索引退一步
                              i-=1;
                             }  
                           }
                         }
                      } 

                    //判断是否是第一次加载蛇身，如果是，则添加蛇头
                    if (snake.snakeBody.length == 0) {
                        var a = {}; 
                        do{
                            a.x = Math.floor(19 * Math.random()) * 10 + snake.box.offsetLeft + parseInt(snake.box.style.borderWidth);
                            a.y = Math.floor(19 * Math.random()) * 10 + snake.box.offsetTop + parseInt(snake.box.style.borderWidth);
                        }while((function(){ 
                                //第一次添加蛇头的时候不能让蛇头挨着边框
                                if(a.x==snake.box.offsetLeft + parseInt(snake.box.style.borderWidth)
                                ||a.x+10==snake.box.offsetLeft + parseInt(snake.box.style.borderWidth)+snake.box.clientWidth
                                ||a.y==snake.box.offsetTop+parseInt(snake.box.style.borderWidth)
                                ||a.y+10==snake.box.offsetTop+parseInt(snake.box.style.borderWidth)+snake.box.clientHeight
                                ){
                                return true;
                                }else{
                                return false;
                                }
                                })());
                        snake.snakeBody.push(a);
                    }
                    //遍历存放蛇的数组并绘制在页面上
                    for (var b in snake.snakeBody) {
                        var d = document.createElement("div");
                        d.style.backgroundColor = 'red'; d.style.border = "0px solid green"; d.style.position = 'absolute'; d.style.zIndex = '1';
                        d.style.width = '10px'; d.style.height = '10px';
                        d.style.left = snake.snakeBody[b].x + "px"; d.style.top = snake.snakeBody[b].y + "px";
                        snake.box.appendChild(d);
                    }
                    return snake;
                },
                eatFood: function () {
                    var foodObj = food.getFood();
                    //判断蛇头是否与食物重合，如果是，则把食物作为蛇头
                    if (
                    snake.snakeBody[snake.snakeBody.length - 1].x == foodObj.offsetLeft
                    && snake.snakeBody[snake.snakeBody.length - 1].y == foodObj.offsetTop
                    ) {
                        var o = {};
                        switch (snake.fangxiang) {
                            case "left":
                                o.x = snake.snakeBody[snake.snakeBody.length - 1].x - 10;
                                o.y = snake.snakeBody[snake.snakeBody.length - 1].y;
                                break;
                            case "right":
                                o.x = snake.snakeBody[snake.snakeBody.length - 1].x + 10;
                                o.y = snake.snakeBody[snake.snakeBody.length - 1].y;
                                break;
                            case "up":
                                o.x = snake.snakeBody[snake.snakeBody.length - 1].x;
                                o.y = snake.snakeBody[snake.snakeBody.length - 1].y - 10;
                                break;
                            case "down":
                                o.x = snake.snakeBody[snake.snakeBody.length - 1].x;
                                o.y = snake.snakeBody[snake.snakeBody.length - 1].y + 10;
                                break;
                            default:
                                break;
                        }

                        //把原来的食物删除
                        snake.box.removeChild(foodObj);
                        //把吃到的食物加到蛇头
                        snake.snakeBody.push(o);
                        //重新绘制食物
                        food.foodMake();
                    }
                }
            }

            var food = {
                //生成食物
                foodMake: function () {
                    //定义食物出现的位置
                    var x, y;
                    do {
                        x = Math.floor(19 * Math.random()) * 10 + snake.box.offsetLeft + parseInt(snake.box.style.borderWidth);
                        y = Math.floor(19 * Math.random()) * 10 + snake.box.offsetTop + parseInt(snake.box.style.borderWidth);
                    } while ((function () {
                        //蛇身是否在该位置，如果蛇身和食物重合则重新生成食物
                        var t = false;
                        for (var o in snake.snakeBody) {
                            t = snake.snakeBody[o].x == x && snake.snakeBody[o].y == y
                            if (t) {
                                break;
                            }
                        }
                        return t;
                    })());
                    var d = document.createElement("div");
                    d.style.backgroundColor = 'red'; d.style.border = "0px solid green"; d.style.position = 'absolute'; d.style.zIndex = '1';
                    d.style.width = '10px'; d.style.height = '10px'; d.setAttribute("id", "foodDiv"); //设置食物的ID用于区别蛇身
                    d.style.top = x + "px"; d.style.left = y + "px";
                    return snake.box.appendChild(d); //返回新生成的食物
                },
                //得到食物
                getFood: function () {
                    return document.getElementById("foodDiv");
                }
            }
            //先绘制食物
            food.foodMake();
            //绘制并移动蛇身
            snake.drawSnake().move();
            //控制方向键
            document.onkeydown = function (event) {
                var e = event || window.event; 
                switch (e.keyCode) {
                    case 37:
                        if (snake.fangxiang != "right") {
                            snake.fangxiang = "left";
                        }
                        break;
                    case 38:
                        if (snake.fangxiang != "down") {
                            snake.fangxiang = "up";
                        }
                        break;
                    case 39:
                        if (snake.fangxiang != "left") {
                            snake.fangxiang = "right";
                        }
                        break;
                    case 40:
                        if (snake.fangxiang != "up") {
                            snake.fangxiang = "down";
                        }
                    default:
                        break;
                }
            }
        }

    </script>
</head>
<body style='text-align: center'>
    <div id='snakeBox' style='border: 1px solid black; width: 200px; height: 200px'>
    </div>
</body>
</html>
```