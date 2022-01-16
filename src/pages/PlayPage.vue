<script lang="ts" setup>
import { reactive } from "vue";
import { Tetromino, TETROMINO_TYPE } from '../common/Tetromino';
import { Field } from '../common/Field';

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
});

// マス目の状態を元に対応したテトリミノの識別子 (クラス名) を取得する
const classBlockColor = (_x: number, _y: number): string => {
  const type = tetris.field.data[_y][_x];
  if(type > 0) {
    return Tetromino.id(type as TETROMINO_TYPE);
  }

// テトリスのフィールドのマス目が空白であれば、
// 落下中のテトリミノの描画範囲をチェックする
  const { x, y } = tetromino.position;
  const { data } = tetromino.current;

// 落下中のテトリミノの描画範囲のマス目をチェックする
// マス目が空白以外であれば、対応するテトリミノの識別子を返却する
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

// テトリスの落下動作を記述
setInterval(() => {
  tetris.field = Field.deepCopy(staticField);

  tetromino.position.y++;
  tetris.field.update(tetromino.current.data, tetromino.position);
}, 1 * 1000);
tetris.field.update(tetromino.current.data, tetromino.position);

</script>

<template>
  <h1>プレイ画面</h1>
  <h2>ユーザ名: {{ $route.query.name }}</h2>

  <div class="container">
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
</style>