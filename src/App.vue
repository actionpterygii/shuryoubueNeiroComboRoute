<script setup>
import {ref, computed} from "vue";

import effects_data from "./data/effects.json";
import melodies_data from "./data/melodies.json";
import horns_data from "./data/horns.json";
import tones_data from "./data/tones.json";

// 音色変換用
const score_map = {
  "1": "♩",
  "2": "♪",
  "3": "♫"
};

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
  added_effects.value.push({
    ...effect,
    // 音符並びHTMLも追加
    score_html: event.currentTarget.querySelector("span").innerHTML
  });
};
// 譜面から削除
const removeEffect = (index) => {
  added_effects.value.splice(index, 1);
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
      <select id="horn_select" v-model="selected_horn_id">
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
        <button
          @click="removeEffect(index)"
          class="remove_effect_button"
        >
          ×
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
.added_effects div button {
  position: absolute;
  top: -15px;
  right: 0px;
  background: none;
  cursor: pointer;
  line-height: 16px;
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
</style>
