<script setup>
import {ref, computed} from "vue";

import effects_data from "./data/effects.json";
import melodies_data from "./data/melodies.json";
import horns_data from "./data/horns.json";
import tones_data from "./data/tones.json";

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

// 
const formatSelectionHornHTML = ((horn) => {
  const colored_tone = horn.tone_ja.split("")
    .map((char, index) => {
      const color = horn.tone_color[index];
      return `<span style="color: ${color}">${char}</span>`;
    })
    .join("");
  return `${horn.name} (${colored_tone})`;
});

// 選択中の狩猟笛id
const selected_horn_id = ref(null);
// 選択中の狩猟笛
const selected_horn = computed(() => hornes.find(horn => horn.id == selected_horn_id.value) || null);
// 譜面の生成
const formatScore = (score, tone_colors) => {
  const score_map = {
    "1": "A",
    "2": "B",
    "3": "C"
  };
  return score.split("").map((char, index) => `<span style="color: ${tone_colors[parseInt(char) - 1]}">${score_map[char]}</span>`).join("");
};
</script>

<template>
  <main>
    <pre>
      <!-- {{ hornes }} -->
    </pre>
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
      >
        <li
          v-for="(effect, index) in selected_horn.melody.effects"
          :key="index"
          :data-effect-type="effect.effect_type"
        >
          {{ effect.effect_name }} {{ formatScore(effect.score, selected_horn.tone_colors) }}
        </li>
      </ul>
    </div>
  </main>
</template>

<style scoped>

</style>
