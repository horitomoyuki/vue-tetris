<script lang="ts" setup>
import { reactive } from "vue";
import { Tetromino, TETROMINO_TYPE } from '../common/Tetromino';
import { Field } from '../common/Field';
import TetrominoPreviewComponent from '../components/TetrominoPreviewComponent.vue'

// フィールドを保持する
let staticField = new Field();

// テトリミノ落下中のテトリスのフィールドを保持
const tetris = reactive({
  field: new Field(),
});
// 落下中のテトリミノの状態を保持する
const tetromino = reactive({
  current: Tetromino.random(),
  position: { x: 3, y: 0 },
  rotate: 0,
  next: Tetromino.random(),
});

const currentTetrominoData = () => {
  return Tetromino.rotate(tetromino.rotate, tetromino.current.data);
}

const classBlockColor = (_x: number, _y: number) => {
  const type = tetris.field.data[_y][_x]
  if(type > 0) {
    return Tetromino.id(type as TETROMINO_TYPE)
  }
  const { x, y } = tetromino.position
  const data = currentTetrominoData();

  if (y <= _y && _y < y + data.length) {
    const cols = data[_y - y];
    if (x <= _x && _x < x + cols.length) {
      if (cols[_x - x] > 0) {
        return Tetromino.id(cols[_x - x] as TETROMINO_TYPE);
      }
    }
  }
  return "";
}

// マス目の状態を元に対応したテトリミノの識別子 (クラス名) を取得する
const canDropCurrentTetromino = (): boolean => {
  const { x, y } = tetromino.position;
  const droppedPosition = {x, y: y + 1};

  const data = currentTetrominoData();
  return tetris.field.canMove(data, droppedPosition);
}

const nextTetrisField = () => {
  const data = currentTetrominoData();
  const position = tetromino.position;

  tetris.field.update(data, position);

  staticField = new Field(tetris.field.data);
  tetris.field = Field.deepCopy(staticField);

  tetromino.current = tetromino.next;
  tetromino.next = Tetromino.random();
  tetromino.rotate = 0;
  tetromino.position = { x: 3, y: 0 };
}

const onKeyDown = (e: KeyboardEvent) => {
  switch (e.key) {
    case " ": {
      const nextRotate = (tetromino.rotate + 1) % 4;
      const data = Tetromino.rotate(nextRotate, tetromino.current.data)
      if (tetris.field.canMove(data, tetromino.position)) {
        tetromino.rotate = nextRotate
      }
    }
    break;
    case "Down":
    case "ArrowDown":
      if(canDropCurrentTetromino()) {
        tetromino.position.y++;

        // resetDrop 関数で矢印キーで下に移動した直後は、
        // 1秒ごとに 1マス下に落下する処理をリセットして
        // 一気に 2マス下に移動してしまう事象を防ぐ
        resetDrop();
      } else {
        nextTetrisField();
      }
    break;
    case "Up":
    case "ArrowUp":
      while(canDropCurrentTetromino()) {
        tetromino.position.y++;
        resetDrop();
      }
    nextTetrisField();
    break;
    case "Left":
    case "ArrowLeft": {
      const data = currentTetrominoData()
      const { x, y } = tetromino.position
      const leftPosition = {x: x - 1, y};
      if(tetris.field.canMove(data, leftPosition)) {
        tetromino.position.x--;
      }
    }
    break;
    case "Right":
    case "ArrowRight": {
      const data = currentTetrominoData()
      const { x, y } = tetromino.position
      const rightPosition = {x: x + 1, y};
      if(tetris.field.canMove(data, rightPosition)) {
        tetromino.position.x++;
      }
    }
  }
}

// 1 秒ごとに 1 マス下に落下する関数を作成する関数
const resetDropInterval = () => {
  // 1秒ごとに発火する関数の識別子
  let intervalId = -1;

  return () => {
    // 1 秒ごとに発火する処理が既に実行済みの場合は、
    // その処理を破棄することで 1 秒ごとに 1 マス下に落下する処理を一旦解除する
    if (intervalId !== -1) clearInterval(intervalId);

    // 1 秒ごとに 1 マス下に落下する処理を再び発火させる
    // `intervalId` には 1 秒ごとに発火する処理の識別子を格納する
    intervalId = setInterval(() => {
      tetris.field = Field.deepCopy(staticField);

      if(canDropCurrentTetromino()) {
        tetromino.position.y++;
      } else {
        nextTetrisField();
      }
    }, 1 * 1000);
  };
};

// 1 秒ごとに 1 マス下に落下する処理をリセットする関数を作成する
const resetDrop = resetDropInterval();

// プレイページの呼び出し直後に 1 秒ごとに 1 マス下に落下する処理を発火させる
resetDrop();

</script>

<template>
  <h1>プレイ画面</h1>
  <h2>ユーザ名: {{ $route.query.name }}</h2>

  <div class="container">
    <div class="tetris">
      <table class="field" style="border-collapse: collapse">
        <tr v-for="(row, y) in tetris.field.data" :key="y">
          <!-- テトリスのフィールドの各マス目にその状態を描画する (0: 空白, 1: I-テトリミノ, etc.) -->
          <td
            class="block"
            v-for="(col, x) in row"
            :key="() => `${x}${y}`"
            :class="classBlockColor(x, y)"
          />
        </tr>
      </table>
    </div>
    <div class="information">
      <TetrominoPreviewComponent v-bind:tetromino="tetromino.next.data" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .container {
    display: flex;
    justify-content: center;
    align-items: stretch;
  }
  
  .field {
    border: ridge 0.4em #2c3e50;
    border-top: none;
  }
  
  .block {
    width: 1em;
    height: 1em;
    border: 0.1px solid #95a5a6;
  /*
    各テトリミノに対応した色を扱うクラス定義
    .block-i, .block-o のようにクラスが定義される
  */
    &-i {
    background: #3498db;
    }
    &-o {
      background: #f1c40f;
    }
    &-t {
      background: #9b59b6;
    }
    &-j {
      background: #1e3799;
    }
    &-l {
      background: #e67e22;
    }
    &-s {
      background: #2ecc71;
    }
    &-z {
      background: #e74c3c;
    }
  }

  /** テトリスに関する情報をテトリスのフィールドの右に表示する **/
  .information {
    margin-left: 0.5em;
  }
</style>