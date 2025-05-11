<script setup>
import {ref, computed} from "vue";

import effects_data from "./data/effects.json";
import melodies_data from "./data/melodies.json";
import horns_data from "./data/horns.json";
import tones_data from "./data/tones.json";
import score_map from "./data/score_map.json";


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
// 選択中の譜面
const added_effects = ref([]);

// 譜面に追加
const addEffects = (event, effect) => {
  // 追加する旋律効果
  let add_effect = { ...effect };
  // 音符並びHTML要素を取得
  let score_html = event.currentTarget.querySelector("span").innerHTML;
  console.log(added_effects.value.length);
  // 最初じゃない場合
  if (added_effects.value.length > 0) {
    // 既存の最後のスコア
    const last_score_fc = added_effects.value[added_effects.value.length - 1].score;
    console.log(last_score_fc);
    // 今回追加するスコア
    const add_score_fc = add_effect.score;
    console.log(add_effect.score);
    console.log(add_score_fc);

    // 一致位置（ない場合は0）
    let position = 0;
    // 今回追加を後ろから前に移動させて一致箇所を探す
    for (let i = 0; i <= add_score_fc.length; i++) {
      const sub_last_score_fc = last_score_fc.slice(-i);
      const sub_add_score_fc = add_score_fc.slice(0, i);
      console.log('naka');
      console.log(sub_last_score_fc);

      // 一致したら終了(既存の最後と完全一致はNG)
      if (sub_last_score_fc === sub_add_score_fc && !(last_score_fc === add_score_fc && sub_last_score_fc === last_score_fc && sub_add_score_fc === add_score_fc)) {
        position = i;
        console.log('owa');
        console.log(position);
        break;
      }
    }
    console.log(position);
    // 一致ある場合
    if (position > 0) {
      // スコアを短くしたやつで上書き
      add_effect.score = add_effect.score.slice(position);
      console.log(add_effect.score);

      // 音符並びHTMLも短く
      score_html = score_html.replace(/<span[^>]*>.*?<\/span>/g, (match) => {
        console.log(match);
        if (position > 0) {
          position--;
          // 置き換え
          return '';
        }
        return match;
      });
      console.log(score_html);

    }
  }
  console.log(add_effect.score);

  added_effects.value.push({
    ...add_effect,
    score_html: score_html
  });
};
// 譜面から全削除
const removeAllEffect = () => {
  added_effects.value = [];
  // todo;ボタンのやつも全削除
};

// 譜面の生成
const formatScore = (score, tone_colors) => {
  return score.split("").map((char) => `<span style="color: ${tone_colors[parseInt(char) - 1]}">${score_map[char]}</span>`).join("");
};
</script>

<template>
  <main>
    <div>
      <label for="horn_select">狩猟笛を選択</label>
      <select
        v-model="selected_horn_id"
        id="horn_select"
      >
       <!-- todo;選択時譜面とか全部クリア -->
        <option
          v-for="horn in hornes"
          :key="horn.id"
          :value="horn.id"
        >
          {{ `${horn.name} (${horn.tone_ja})` }}
        </option>
      </select>
    </div>
    <div>
      <ul
        v-if="selected_horn"
        class="effect_list"
      >
        <li
          v-for="(effect, index) in selected_horn.melody.effects"
          :key="index"
          @click="addEffects($event, effect)"
        >
          {{ effect.effect_name }}
          <span
            v-html="formatScore(effect.score, selected_horn.tone_colors)"
            class="effect_list__score"
          ></span>
        </li>
      </ul>
    </div>
    <div class="added_effects">
      <div
        v-for="(added_effect, index) in added_effects"
        :key="index"
      >
        <div>
          <p>{{ added_effect.effect_name }}</p>
          <div
            v-html="added_effect.score_html"
          ></div>
        </div>
      </div>
    </div>
    <button
      v-if="added_effects.length"
      @click="removeAllEffect()"
      class="remove_all_effect_button"
    >
      ×
    </button>
  </main>
</template>

<style>
main {
  color: #333333;
  margin: 10px;
}

.effect_list {
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  gap: 10px;
}
.effect_list li {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 5px 10px;
  border: 1px solid #dddddd;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
}
.effect_list li:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.effect_list__score {
  font-size: 18px;
}

.added_effects {
  margin-top: 20px;
  display: flex;
  overflow-x: auto;
  white-space: nowrap;
  padding-top: 20px;
}
.added_effects div {
  position: relative;
}
.added_effects div div {
}
.added_effects div div div {

}
.added_effects div div p {
  font-size: 10px;
  text-align: center;
}
.added_effects div div div span {
  font-size: 35px;
}
.remove_all_effect_button {
  cursor: pointer;
  line-height: 16px;
}
</style>
