<template>
  <div>
    <div class="container">
      <div class="game-over-banner" v-show="gameOver">
        <div class="game-over-wrapper">
          <p>Game over :(</p>
          <span class="score">SCORE: {{score}}</span>
          <button @click="newGame">NEW GAME</button>
        </div>
      </div>
      <div class="row justify-content-center" :class="{'game-over': gameOver}">
        <div class="col-md-12">

          <span class="score">SCORE: {{score}}</span>
        </div>
      </div>
      <div class="row justify-content-center">
        <div class="col-md-6 col-lg-4 col-9">
          <div class="drop-row">
            <div v-for="(item,index) in boardRow1">
              <drop v class="drop" :class="{'activeRow': item.value != 0  }"
                @dragover="handleDragover(item, 1, ...arguments)" @drop="handleDrop">
                <div v-if="item.value != 0">
                  <p> {{item.value}}</p>
                </div>
              </drop>
            </div>
            <div v-for="(item,index) in boardRow2">
              <drop v class="drop" :class="{'activeRow': item.value != 0  }"
                @dragover="handleDragover(item, 2, ...arguments)" @drop="handleDrop">
                <div v-if="item.value != 0">
                  <p> {{item.value  }}</p>
                </div>
              </drop>
            </div>
            <div v-for="(item,index) in boardRow3">
              <drop v class="drop" :class="{'activeRow': item.value != 0  }"
                @dragover="handleDragover(item, 3, ...arguments)" @drop="handleDrop">
                <div v-if="item.value != 0">
                  <p> {{item.value}}</p>
                </div>
              </drop>
              </di>
            </div>
          </div>
        </div>
      </div>
      <div class="row justify-content-center" style="margin-top: 38px;">
        <div class="col-md-4 col-12 d-flex justify-content-end">
          <div class="inserting-row" >
            <div v-for="(number,index) in arrayOfNumbersForPlay">
              <div class="drag" v-if="index < 2">
                <drag class="drag-box cursor-not-allowed" :transfer-data="{ value:number,index:index }" :draggable=false
                  @dragend="dragging = null">
                  <span class="number">{{ number }}</span>
                </drag>
              </div>
              <div class="drag-allowed " v-else-if="index == 2">
                <drag class=" cursor-drag drag-box-allowed" :transfer-data="{ value:number,index:index }"
                  :effect-allowed="['move']" @dragstart="dragStartNumber" @dragend="dragEndNumber">
                  <span class="number">{{ number }}</span>
                </drag>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-2">
          <div class="keep-number-row">
            <span>Keep</span>
            <div class="keep-number-wrapper">
              <drop class="" @dragover="handleDragoverKeepNumber(...arguments)"
                @drop="handleDropKeepNumber( ...arguments)">
                <drag class="drop-keep-number" @dragstart="keepNumberDragStartMethod(...arguments)"
                  @dragend="keepNumberDragEndMethod" :transfer-data="{ number: keepNumber }"
                  :draggable="!keepNumber == 0">
                  <p v-if="keepNumber != 0" class="keep-number">{{ keepNumber }}</p>
                </drag>
              </drop>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  </div>
  </div>
</template>

<script>
  import draggable from "vuedraggable";
  import $ from 'jquery'
  export default {
    data() {
      return {
        arrayOfNumbersForPlay: [],
        gameOver: false,
        keepNumber: 0,
        keepNumberDragStart: false,
        boardRow1: [],
        boardRow2: [],
        boardRow3: [],
        currentDragedIn: 0,
        currentRowNumber: 0,
        scoreAddition: 0,
        score: 0,
        draggable: [
          {
            value: 1
          }
        ]
      }
    },
    created() {
      this.initializeGameBoard()
    },
    methods: {
      dragStartNumber() {
        setTimeout(() => {
          $('.drag-box-allowed .number').css('visibility', 'hidden')
          $('.drag-box-allowed').css('background-color', 'transparent')
        }, 0)
      },
      dragEndNumber() {
        setTimeout(() => {
          $('.drag-box-allowed .number').css('visibility', 'visible')
          $('.drag-box-allowed').css('background-color', '#1BD9B3')
        }, 0)
      },
      handleDragover(rowData, rowNumber, data, event) {
        if (rowData.positionFilled) {
          event.dataTransfer.dropEffect = 'none';
        } else {

          this.currentDragedIn = rowData.index;
          this.currentRowNumber = rowNumber;
        }
      },
      handleDrop(data) {

        let currentRow = this.resolveRow(this.currentRowNumber);
        let filledIndex = this.currentDragedIn;

        if (!this.keepNumberDragStart) {
          this.arrayOfNumbersForPlay.indexOf(data.value) > -1 ? this.arrayOfNumbersForPlay.splice(this.arrayOfNumbersForPlay.indexOf(data.value), 1) : false
          this.arrayOfNumbersForPlay.unshift(Math.floor((Math.random() * 24) + 2));
        } else {
          data.value = data.number;
          this.keepNumber = 0;
        }


        currentRow[this.currentDragedIn].value = data.value;
        currentRow[this.currentDragedIn].positionFilled = true;

        this.numbersCalculation(currentRow, filledIndex);
        this.checkForEmptyFields();

      },
      handleDropKeepNumber(data) {
        this.keepNumber = data.value;
        $('.drop-keep-number').css('background-color', '#1BD9B3')
        this.arrayOfNumbersForPlay.splice(-1, 1)
        this.arrayOfNumbersForPlay.unshift(Math.floor((Math.random() * 24) + 2));
      },
      keepNumberDragStartMethod(data, event) {
        this.keepNumberDragStart = true;
        setTimeout(() => {
          $('div >.keep-number').css('visibility', 'hidden')
          $('.drop-keep-number').css('background-color', 'transparent')
        }, 0)
      },
      keepNumberDragEndMethod() {
        this.keepNumberDragStart = false;
        $('div >.keep-number').css('visibility', 'visible')
      },
      handleDragoverKeepNumber(data, event) {
        if (this.keepNumber != 0) {
          event.dataTransfer.dropEffect = 'none';
        }
      },
      //Function that resolves which row is being targeted
      resolveRow(rowNumber) {
        let obj;
        switch (rowNumber) {
          case 1:
            obj = this.boardRow1;
            break;
          case 2:
            obj = this.boardRow2;
            break;
          case 3:
            obj = this.boardRow3;
            break;
        }
        return obj;
      },
      //Recursive function that divides two numbers based on their position in board row
      numbersCalculation(currentRow, filledIndex) {
        let filledIndexAfterDivision;
        //Condition which passes only if previos number in the row is not undefined(if there is a field in row) or if it is not 0(empty)
        if (currentRow[filledIndex - 1] != undefined && currentRow[filledIndex - 1].value != 0) {
          let divisionResult = currentRow[filledIndex - 1].value / currentRow[filledIndex].value;
           //Condition which passes only if the result of division beetwen two number is rounded
          if (divisionResult % 1 == 0) {
            this.score = this.score + currentRow[filledIndex].value * 2;
            currentRow[filledIndex].value = divisionResult;
            currentRow[filledIndex - 1].value = 0;
            currentRow[filledIndex - 1].positionFilled = false;
            filledIndexAfterDivision = currentRow[filledIndex].index
          }
        }
         //Condition which passes only if next number in the row is not undefined(if there is a field in row) or if it is not 0(empty)
        if (currentRow[filledIndex + 1] != undefined && currentRow[filledIndex + 1].value != 0) {
          let divisionResult = currentRow[filledIndex].value / currentRow[filledIndex + 1].value;
           //Condition which passes only if the result of division beetwen two number is rounded
          if (divisionResult % 1 == 0) {
            this.score = this.score + currentRow[filledIndex + 1].value * 2;
            currentRow[filledIndex + 1].value = divisionResult;
            currentRow[filledIndex].value = 0;
            currentRow[filledIndex].positionFilled = false;
            filledIndexAfterDivision = currentRow[filledIndex + 1].index
          }
        }
        //Contition that check if there is existing field before or after one of two division operations above,if there is the numbersCalculation() will be called again 
        if (filledIndexAfterDivision != undefined) {
          this.numbersCalculation(currentRow, filledIndexAfterDivision)
        }
      },
      //Function that check if there is still empty fields in board ,if not the game is over 
      checkForEmptyFields() {
        let joinedRows = [...this.boardRow1, ...this.boardRow2, ...this.boardRow3];
        let result = joinedRows.filter(item => item.positionFilled == false);
        if (result.length == 0) {
          this.gameOver = true;
        }
      },
      //Function that initializes new game
      newGame() {
        Object.assign(this.$data, this.$options.data());
        this.initializeGameBoard();
        $('.drag-box-allowed').css('background-color', '#1BD9B3')
      },
      //Function that insert new random number in arrayOfNumbersForPlay array
      fillNumberRow() {
        for (let i = 0; i <= 3; i++) {
          this.arrayOfNumbersForPlay.push(Math.floor(Math.random() * (24 - 2 + 1)) + 2);
        }
      },
      //Function that creates game board at begining of a new game
      initializeGameBoard(){
        for(let i = 0;i < 3;i++){
          this.boardRow1.push({
          index: i,
          value: 0,
          positionFilled: false
          })
        }
        for(let i = 0;i < 3;i++){
          this.boardRow2.push({
          index: i,
          value: 0,
          positionFilled: false
          })
        }
        for(let i = 0;i < 3;i++){
          this.boardRow3.push({
          index: i,
          value: 0,
          positionFilled: false
          })
        }
        $('.drag-box-allowed').css('background-color', 'transparent')
        this.fillNumberRow();
      }
    }
  }

</script>

<style lang="scss">
  body {
    background-color: #383838;
  }

  .number {
    font-size: 40px;
    color: white;

  }

  .activeRow {
    background-color: #1BD9B3;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .activeRow p {
    font-size: 36px;
    margin: 0;
    color: white;
  }

  .drag-box {
    width: 80px;
    height: 80px;
    opacity: 0.7;
    border: none;
    background-color: #1BD9B3;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 5px;
  }

  .drag-box-allowed {
    width: 90px;
    height: 90px;
    background-color: #1BD9B3;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 5px;
  }

  .drag {
    width: 80px;
    height: 80px;

    background-color: #727272;
    border-radius: 5px;
    margin-right: 20px;
  }

  .drag-allowed {
    width: 90px;
    height: 90px;
    background-color: #727272;
    border-radius: 5px;

  }

  .drop {
    margin: 1px;
    width: 90px;
    height: 90px;
    border: 1px solid #383838;
    background-color: #727272;
    border-radius: 5px;
  }

  .activeRow {
    background-color: #1BD9B3;
    display: flex;
    justify-content: center;
    align-items: center;
  }



  .drop-row {
    display: flex;
    flex-wrap: wrap
  }

  .inserting-row {
    display: flex;
    margin-top: 20px;
    align-items: center;
  }

  .game-over-banner {
    position: absolute;
    top:0;
    color: white;
    font-size: 30px;
    left: 21px;
    position: absolute;
    width: 100%;
    left: 0;
    width: 100%;
    text-align: center;
    left: 0;
    z-index: 22;
    background-color: #727272;
    height:100vh;
    opacity: 1;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .game-over-wrapper{
    width: 582px;
    height: 373px;
    padding: 20px;
    border: 1px solid black;
  }
  .game-over-wrapper span{
  padding:0;
  }
  .game-over-banner button{
    background-color: #1BD9B3;
    border-radius: 4px;
    border:none;
    outline:none;
    cursor: pointer;
    width: 300px;
    color: white;
    height: 64px;
  }
  .keep-number-row{
    padding-top: 13px;
  }
.keep-number-row span{
font-size: 30px;
    position: absolute;
    color: white;
    top: -32px;
    left: 36px;
}
  .keep-number-wrapper {
    width: 110px;
    height: 110px;
    background-color: #727272;
    border-radius: 5px;
  }

  .drop-keep-number {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 110px;
    height: 110px;
    /* background-color: #1BD9B3; */
    border-radius: 5px;
  }

  .drop-keep-number p {
    font-size: 50px;
    color: white;
  }

  .cursor-not-allowed {
    cursor: not-allowed;
  }

  .cursor-drag {
    cursor: drag;
  }

  .score {
    font-size: 50px;
    color: white;
    text-align: center;
    display: block;
    padding-right: 79px;
    margin: 50px 0 80px 0;
  }
</style>