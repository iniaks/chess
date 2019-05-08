<template>
    <div>
        <div class='chess-board'>
            <div v-for='(row, y) in chessboard' :key='y' class='chess-board__row'>
                <div v-for='(cell, x) in row' :key='x' class='chess-board__cell'>
                    <!-- {{cell.x}},{{cell.y}} -->
                    <div class='mask' v-if='selected_unit' :style='{opacity: cell.movable ? 0 : 0.25}' @click.stop='tap(cell)'/>
                    <div class='chess-board__piece'
                    v-if='cell.unit'
                    :style='{backgroundColor: cell.unit.camp, backgroundImage: "url(" + require(`../../assets/${cell.unit.icon}`) + ")"}' @click.stop='choose(cell)'/>
                </div>
            </div>
        </div>

        <div class='chess-dev'>
            x: <input v-model='test_x'/>
            <br>
            y: <input v-model="test_y"/>
            <br>
            阵营: <input v-model="test_camp"/>
            <br>
            <button @click='createKing'>生成王</button>

            <p>选择棋子：{{selected_unit ? selected_unit.unit.name : '未选择'}}</p>
        
            <p>移动记录</p>
            <div v-for='(record, index) in move_records' :key='index' class='move-record'>{{record}}</div>

            <!-- <p>阵亡列表</p>
            <div v-for='(piece, index) in dead_list' :key='index' class='dead-list'>{{piece.camp}} {{piece.name}}</div> -->
        </div>
    </div>
</template>

<script>
    class Cell {
        x
        y
        unit = null
        selected = false
        movable = false
        constructor (x, y) {
            this.x = x
            this.y = y
        }
        putDown (chess) {
            this.unit = chess
        }
        putUp () {
            this.unit = null
        }
        checkMovable (scopes, camp) {
            let movable = false
            const self = this
            scopes.forEach(scope => {
                if (scope[0] == self.x && scope[1] == self.y && (!self.unit || self.unit.camp != camp)) movable = true
            })
            this.movable = movable
        }
    }

    class Chess {
        name
        icon
        scope
        camp
        constructor (name, icon, camp, scope) {
            this.name = name
            this.icon = icon
            this.camp = camp
            this.scope = scope
        }
    }

    export default {
        data () {
            return {
                test_x: 1,
                test_y: 1,
                test_camp: '#66D4C3',
                size: 18,
                chessboard: [],
                selecting: false,
                selected_unit: null,
                move_records: [],
                dead_list: []
            }
        },
        methods: {
            createKing () {
                let king = new Chess('king', 'king-chess-piece-shape.png', this.test_camp, (x, y) => {
                    let scopes = [
                        [x, y],
                        [x - 1, y],
                        [x + 1, y],
                        [x, y - 1],
                        [x, y + 1]
                    ]
                    scopes.forEach((scope, index) => {
                        if (scope[0] < 1 || scope[0] > 18 || scope[1] < 1 || scope[1] > 18) scopes.splice(index, 1)
                    })
                    return scopes
                })
                this.chessboard[this.test_x - 1][this.test_y - 1].putDown(king)
            },
            choose (cell) {
                // const self = this
                // selecting checkpiece
                this.selecting = !this.selecting
                this.selected_unit = this.selecting ? cell: null

                if (this.selected_unit) {
                    this.chessboard.forEach(row => {
                        row.forEach(item => {
                            item.checkMovable(cell.unit.scope(cell.x, cell.y), cell.unit.camp)
                        })
                    })
                }
            },
            tap (cell) {
                console.log(cell)
                if (cell.movable) {
                    this.move_records.push(`${this.selected_unit.unit.camp} ${this.selected_unit.unit.name} (${this.selected_unit.x},${this.selected_unit.y}) => (${cell.x},${cell.y})`)
                    if (cell.unit && cell.unit.camp != this.selected_unit.unit.camp) {
                        this.dead_list.push(cell.unit)
                    }
                    cell.putDown(this.selected_unit.unit)
                    this.selected_unit.putUp()
                }
                
                this.selecting = false
                this.selected_unit = null
            },
            init () {
                for (let lat = 1; lat <= this.size; lat++) {
                    let row = []
                    for (let lng = 1; lng <= this.size; lng++) {
                        let cell = new Cell(lng, lat)
                        row.push(cell)
                    }
                    this.chessboard.push(row)
                }
                // console.log(this.chessboard)
            }
        },
        mounted () {
            this.init()
        }
    }
</script>


<style lang="less">
    .chess-board {
        float: left;
        width: 100vh;
        height: 100vh;
        .chess-board__row {
            display: flex;
            .chess-board__cell {
                flex: 1;
                height: calc(100vh / 18);
                line-height: calc(100vh / 18);
                text-align: center;
                font-size: 8pt;
                background: #f8f8f8;
                position: relative;
            }
        }
        .mask {
            position: absolute;
            left: 0;
            right: 0;
            background: #000;
            width: 100%;
            height: 100%;
            opacity: .25;
            z-index: 99;
        }
        .chess-board__piece {
            border-radius: 100%;
            position: absolute;
            left: 4px;
            top: 4px;
            width: calc(100% - 8px);
            height: calc(100% - 8px);
            box-shadow: 0 0 1px rgba(0, 0, 0, 0.15);
            background-position: center;
            background-repeat: no-repeat;
            background-size: cover;
        }
        .chess-board__row:nth-child(odd) {
            .chess-board__cell:nth-child(odd) {
                background: #eee;
            }
        }
        .chess-board__row:nth-child(even) {
            .chess-board__cell:nth-child(even) {
                background: #eee;
            }
        }
    }
    .chess-dev {
        float: left;
        padding: 15px;
        input {
            width: 50px;
        }
    }
</style>

