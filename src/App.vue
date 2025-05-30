<script setup>
import {ref, computed} from "vue";

import effects_data from "./data/effects.json";
import melodies_data from "./data/melodies.json";
import horns_data from "./data/horns.json";
import tones_data from "./data/tones.json";
import score_map from "./data/score_map.json";
import commands from "./data/commands.json";


const hornes = horns_data;
hornes.forEach((horn) => {
  // 笛ごとに旋律データを追加
  const melody = melodies_data.filter((melody) => melody.id === horn.melody_id)[0];
  horn.melody = melody;
  // 日本語音色追加
  const tone_ja = horn.melody.tone.map((tone_id) => tones_data.find((tone) => tone.id === tone_id).name).join("");
  horn.tone_ja = tone_ja;
  // 音色色を追加
  const tone_colors = horn.melody.tone.map((tone_id) => tones_data.find((tone) => tone.id === tone_id).color);
  horn.tone_colors = tone_colors;
  // 旋律に旋律効果とタイプを追加
  horn.melody.effects.forEach((effect) => {
    const matching_effect = effects_data.find((effect_data) => effect.effect_id === effect_data.id);
    effect.effect_name = matching_effect.name;
    effect.effect_type = matching_effect.type;
  });
});
 
// 選択中の狩猟笛id
const selected_horn_id = ref(null);
// 選択中の狩猟笛
const selected_horn = computed(() => hornes.find(horn => horn.id == selected_horn_id.value) || null);
// 選択中の効果
const added_effects = ref([]);
// 選択中の譜面
const added_score = ref([]);
// これから見る譜面の最初
const score_pointer = ref(0);
// これから追加するコマンドの音色情報
const next_tone = computed(() => {
  // 見る譜
  const score = added_score.value[score_pointer.value];
  // 色
  const color = selected_horn.value.tone_colors[score - 1];
  // 音符
  const char = score_map[score];
  // 番目
  const number = score_pointer.value + 1;
  // HTML生成
  return `<span class="tag has-text-weight-bold is-size-6" style="color: ${color}">${number}音目${char}</span>`;
});
// コマンドの選択肢
const selectable_commands = computed(() => {
  // 見る譜
  const score = added_score.value[score_pointer.value];
  // 最初の場合
  if (score_pointer.value === 0) {
    return commands.filter(command => command.first === true && command.note === score);
  // 最初以外の場合
  } else {
    // 
    return commands.filter(command => command.second === true && command.note === score);

  }
});
// 選択中のコマンド
const selected_command = ref(null);
// 確定したコマンドたち
const added_commands = ref([]);

// 譜面に追加
const addEffects = (event, effect) => {
  // 追加する旋律効果
  let add_effect = { ...effect };
  // 音符並びHTML要素を取得
  let score_html = event.currentTarget.querySelector("span").innerHTML;
  // 最初じゃない場合
  if (added_score.value.length > 0) {
    // 既存のスコア
    const score_fc = added_score.value.join("");
    // 今回追加するスコア
    const add_score_fc = add_effect.score;
    // 一致位置（ない場合は0）
    let position = 0;
    // 今回追加を後ろから前に移動させて一致箇所を探す
    // 完全一致をしないための-1
    for (let i = 0; i <= add_score_fc.length - 1; i++) {
      const sub_score_fc = score_fc.slice(-i);
      const sub_add_score_fc = add_score_fc.slice(0, i);
      // 一致したら終了
      if (sub_score_fc === sub_add_score_fc) {
        position = i;
        break;
      }
    }
    // 一致ある場合
    if (position > 0) {
      // スコアを短くしたやつで上書き
      add_effect.score = add_effect.score.slice(position);
      // 音符並びHTMLも短く
      score_html = score_html.replace(/<span[^>]*>.*?<\/span>/g, (match) => {
        if (position > 0) {
          position--;
          // 音符置き換え
          return '';
        }
        return match;
      });
    }
  }
  added_score.value.push(...[...add_effect.score].map(Number));
  added_effects.value.push({
    ...add_effect,
    score_html: score_html
  });
};
// コマンド追加
const addCommand = () => {
  const command_id = selected_command.value;
  const command = commands.find(command => command.id === command_id);
  added_commands.value.push(command);
  ++score_pointer.value;
  selected_command.value = null;
};
// 譜面から全削除
const removeAllEffect = () => {
  added_effects.value = [];
  added_score.value = [];
  score_pointer.value = 0;
  selected_command.value = null;
  added_commands.value = [];
};
// コマンドを一つ削除
const removeCommand = () => {
   --score_pointer.value;
  added_commands.value.pop();
};

// 譜面の生成
const formatScore = (score, tone_colors) => {
  return score.split("").map((char) => `<span style="color: ${tone_colors[parseInt(char) - 1]}">${score_map[char]}</span>`).join("");
};
</script>

<template>
  <main>
    <div class="block mb-6">
      <div class="title is-4">モンハンワイルズ狩猟笛音色コンボルート選定ツール</div>
      <a
        href="https://www.actionpterygii.com/"
        class="is-underlined actionpterygii"
      >actionpterygii</a>
    </div>
    <div class="block">
      <div class="title is-6">①狩猟笛を選択</div>
      <div class="select is-small">
        <select
          @change="removeAllEffect"
          v-model="selected_horn_id"
        >
          <option
            v-for="horn in hornes"
            :key="horn.id"
            :value="horn.id"
          >
            {{ `${horn.name} (${horn.tone_ja})` }}
          </option>
        </select>
      </div>
    </div>
    <div
      v-if="selected_horn"
      class="block"
    >
      <div class="title is-6">②旋律を選択</div>
      <div
        v-if="selected_horn"
        class="buttons are-small"
      >
        <button
          v-for="(effect, index) in selected_horn.melody.effects"
          :key="index"
          @click="addEffects($event, effect)"
          class="button"
        >
          {{ effect.effect_name }}
          <span
            v-html="formatScore(effect.score, selected_horn.tone_colors)"
          ></span>
        </button>
      </div>
    </div>
    <div
      v-if="added_effects.length"
      class="block"
    >
      <div class="box">
        <div class="mb-4">
          選択された旋律
        </div>
        <div class="is-flex added_effects">
          <div
            v-for="(added_effect, index) in added_effects"
            :key="index"
            class="mr-2"
          >
            <p class="is-size-7 has-text-centered">
              {{ added_effect.effect_name }}
            </p>
            <div
              v-html="added_effect.score_html"
              class="is-size-2"
            ></div>
          </div>
        </div>
      </div>
      <button
        @click="removeAllEffect()"
        class="button is-small is-danger is-outlined"
      >
        旋律全削除
      </button>
    </div>
    <div
      v-if="added_effects.length"
      class="block"
    >
      <div class="title is-6">③コマンドを選択</div>
      <div
        v-if="added_score.length !== score_pointer"
        class="is-flex mb-4"
      >
        <div v-html="next_tone"></div>
        <div class="select is-small ml-3">
          <select
            v-if="selectable_commands.length"
            v-model="selected_command"
            @change="addCommand()"
          >
            <option
              v-for="command in selectable_commands"
              :key="command.id"
              :value="command.id"
            >
              {{ command.text }}
            </option>
          </select>
        </div>
      </div>
      <div v-if="added_commands.length">
        <div class="box is-flex is-flex-wrap-wrap">
          <div
            v-for="added_command in added_commands"
            :key="added_command.id"
            class="added_command tag is-large py-5 mr-5 my-2 is-flex is-flex-direction-column"
          >
            <div>
              {{ added_command.button }}
            </div>
            <div class="is-size-7">
              {{ added_command.text }}
            </div>
          </div>
        </div>
        <button
          @click="removeCommand()"
          class="button is-small is-danger is-outlined"
        >
          一つ削除
        </button>
      </div>
    </div>
  </main>
</template>

<style>
main {
  color: #333333;
  margin: 10px;
}

/* 選択された旋律 */
.added_effects {
  overflow-x: auto;
  white-space: nowrap;
}

.added_command {
  position: relative;
}

.added_command:not(:last-child)::after {
  content: "→";
  font-size: 0.75rem;
  display: inline-block;
  position: absolute;
  right: -20px;
  top: 50%;
  transform: translateY(-50%);
}

.actionpterygii {
  color: #33cccc;
}

/* bulma+ */
.is-size-2 span {
  font-size: 2.5rem;
}
</style>
