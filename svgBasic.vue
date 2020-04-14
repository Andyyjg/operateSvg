<template>
    <div style="width: 100%;height: 100%;background: #ffff" class="father">
        <div>
            <button @click="changeStatus('line')">线段</button>
            <button style="margin-left: 50px" @click="changeStatus('arrow')">箭头</button>
            <el-color-picker
                    size="mini"
                    v-model="color"
                    show-alpha
                    @change="chanegColor"
            >
            </el-color-picker>
        </div>
        <div style="width: 100%;height: 80vh;background: #eeeeee;position: relative;margin-top: 50px;overflow: hidden"
             @mousedown="mousedown($event)" id="painitBox">
            <div class="operate" v-for="(item,key) of lines" :key="key">
                <div class="circle" :style="{
                    left: item.x+item.x1+'px',
                    top:item.y+item.y1-15+'px'
                   }"
                     @mousedown="resize($event,'start',key)"
                ></div>
                <div class="circle"
                     @mousedown="resize($event,'end',key)"
                     :style="{
                    left: item.x+item.x2-15+'px',
                    top:item.y+item.y2-15+'px'

                }"></div>
            </div>
            <svg xmlns="http://www.w3.org/2000/svg"
                 style="background: transparent;position: absolute;left: 50px;z-index:10;overflow: visible"
                 :style="{left:item.x+'px',top:item.y+'px'}"
                 v-for="(item,key) of lines"
                 @mousedown="lineDown($event,key)"
                 :key="key"
                 version="1.1">
                <defs>
                    <marker id="arrow"  
                                    markerUnits="strokeWidth"  
                                    markerWidth="12"  
                                    markerHeight="12"  
                                    viewBox="0 0 12 12"  
                                    refX="6"  
                                    refY="6"  
                                    orient="auto">
                               
                        <path d="M2,2 L10,6 L2,10 L6,6 L2,2"  :fill="item.color" />
                    </marker>
                </defs>

                <line

                        class="line"
                        :x1="item.x1" :y1="item.y1"
                        :x2="item.x2" :y2="item.y2"
                        :marker-end="item.type=='line'?'':'url(#arrow)'"
                        @mouseenter.stop="mouseOver($event)"


                         :stroke="item.color" stroke-width="3"/>


                <!--            <div class="front" v-for="(item,key) of lines" :style="{left:item.x1-10+'px',top:item.y1-10+'px'}" v-show="item &&item.x2"  ></div>-->
                <!--            <fiv class="after"></fiv>-->
                <!--             <line :x1="x1" :y1="y1" :x2="x2" :y2="y2"  stroke="#000" stroke-width="3" marker-end="url(#arrow)"/>-->
            </svg>
        </div>


    </div>
</template>

<script>
    export default {
        name: "svgBasic",
        data() {
            return {
                color:'#000',
                x1: 0,
                y1: 0,
                x2: 200,
                y2: 50,
                drawType: -1,
                tmpSon: {x1: 0, x2: 0, y1: 0, y2: 0, type: 'line',color:'#000'},
                draw: false,
                lines: [],
                firstClick: true,
                num: 0,
                moveIndex: -1,
                resizeIndex: -1,
                colorIndex:-1
            }

        },
        methods: {
            mouseOver(e) {

                // e.target.style.cursor
            },
            chanegColor(){
                this.lines[this.colorIndex].color=this.color
            },
            lineDown(e, key) {
                this.color=this.lines[key].color
                this.colorIndex=key
                this.moveIndex = key
                var clientX=0,clientY=0
                const listener = e => {
                    //兼容ie ie下没有movement
                    var movementX=0,movementY=0
                    if(clientX!=0){
                        movementX=e.clientX-clientX
                    }else{
                        movementX=0
                    }
                    if(clientY!=0){
                        movementY=e.clientY-clientY
                    }else{
                        movementX=0
                    }
                    clientX=e.clientX
                    clientY=e.clientY

                    this.lines[this.moveIndex].x += movementX
                    this.lines[this.moveIndex].y += movementY
                    // this.lines[this.moveIndex].x2 += e.movementX
                    // this.lines[this.moveIndex].y2 += e.movementY
                };
                document.body.addEventListener("mousemove", listener);
                document.body.addEventListener("mouseup", () => {
                    document.body.removeEventListener("mousemove", listener);
                    this.moveIndex = -1


                })
            },

            resize(e, status, key) {
                var top = document.getElementById('painitBox').offsetTop,
                    left = document.getElementById('painitBox').offsetLeft
                this.resizeIndex = key
                const listener = event => {
                    // console.log(this.resizeIndex);
                    var tmp = this.lines[this.resizeIndex]
                    if (status == 'start') {
                        this.lines[this.resizeIndex].x1 = event.clientX - tmp.x - left
                        this.lines[this.resizeIndex].y1 = event.clientY - tmp.y - top
                    } else {
                        this.lines[this.resizeIndex].x2 = event.clientX - tmp.x - left
                        this.lines[this.resizeIndex].y2 = event.clientY - tmp.y - top

                    }
                };
                document.body.addEventListener("mousemove", listener);
                document.body.addEventListener("mouseup", () => {
                    document.body.removeEventListener("mousemove", listener);


                })
            },
            mousedown(e) {

                if (this.drawType != -1) {
                    this.firstClick = false
                    this.draw = true
                    // var ele = document.getElementById('svgBox'),w=parseInt(window.getComputedStyle(ele).width),
                    //     h=parseInt(window.getComputedStyle(ele).height)
                    this.tmpSon.x = e.offsetX
                    this.tmpSon.y = e.offsetY
                    this.tmpSon.x1 = 0
                    this.tmpSon.y1 = 0
                    this.tmpSon.type = this.drawType


                    const listener = event => {
                        // console.log(event);
                        var top = document.getElementById('painitBox').offsetTop,
                            left = document.getElementById('painitBox').offsetLeft

                        this.tmpSon.x2 = event.clientX - this.tmpSon.x - left
                        this.tmpSon.y2 = event.clientY - this.tmpSon.y - top
                        if (this.lines[0] && this.num > 0) {
                            this.lines.pop()
                        }

                        this.lines.push(JSON.parse(JSON.stringify(this.tmpSon)))
                        this.num++
                    };
                    document.body.addEventListener("mousemove", listener);
                    document.body.addEventListener("mouseup", () => {
                        document.body.removeEventListener("mousemove", listener);
                        this.drawType = -1
                        this.num = 0


                    })
                } else {
                    this.draw = false
                    this.firstClick = true
                    this.drawType = -1
                    this.num = 0
                }


            },
            mouseup(e) {
                if (this.moveIndex != -1) {
                    this.moveIndex = -1
                }
                // this.draw=false
                // this.drawType=-1
            },
            changeStatus(type) {
                this.drawType = type
            }
        }
    }
</script>

<style scoped lang="scss">
    div.father {
        .line {
            cursor: move;
        }

        .circle {
            background: #0CF2F5;
            height: 40px;
            width: 40px;
            /*border-radius: 50px;*/
            position: absolute;
            z-index: 1200;
            cursor: se-resize;
            /*visibility: hidden;*/
            opacity: 0;
        }

        div.front {
            border: 1px solid red;
            width: 10px;
            height: 10px;
            position: absolute;
        }

        div.after {
            border: 1px solid red;
            width: 10px;
            height: 10px;
            position: absolute;
        }
    }

</style>
