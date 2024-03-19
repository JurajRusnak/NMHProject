<script setup lang="ts">
import {ref, computed, onMounted} from 'vue';
import axios from 'axios'; // Assuming you have axios and Vue.Draggable installed
import draggable from 'vuedraggable';
import WordItem from "@/components/WordItem.vue"; // Import Vue.Draggable

const words = ref<string[]>([]);
const addNewInput = ref<string>('val');

const isLoading = ref(false);
const error = ref<string | null>(null);

const fetchWords = async () => {
  isLoading.value = true;
  error.value = null;
  words.value = [];

  try {
    const response = await axios.get('https://random-word-api.herokuapp.com/word?number=20000&length=15', {
    });

    words.value = response.data;
    words.value = loadWordsFromStorage()

  } catch (err) {
    console.error('Error fetching words:', err);
    error.value = 'Failed to fetch words.';
  } finally {
    isLoading.value = false;
  }
};

const updateLocalStorage = (val: any) => {
  localStorage.setItem(val.moved.element, val.moved.newIndex.toString());
};

const hasWords = computed(() => words.value.length > 0);

const loadWordsFromStorage = (): string[] => {
  let storedWords = [];

  for (let i = 0; i < words.value.length; i++) {
    const word = words.value[i];
    const storedIndex = localStorage.getItem(word);

    if (storedIndex) {
      const parsedIndex = parseInt(storedIndex);
      storedWords[parsedIndex] = word

    } else {
      if(storedWords.findIndex(x => !x) === -1)  {
        storedWords.push(word)
      }
      else {
        storedWords[storedWords.findIndex(x => !x)] = word

      }
    }
  }

  return storedWords.filter(x => x) // Filter out any empty values
};

const addNewWord = () => {
  words.value.push(addNewInput.value)
}

const removeWord = (word) => {
  words.value.splice(words.value.indexOf(word), 1)
}

onMounted(() => {
  fetchWords();
})

</script>

<template>
  <div class="grid-cols-2 grid">
    <div class="mb-8"><button @click="fetchWords">Refresh</button></div>
    <div class="flex ">
      <div class=""> <button class="h-8 rounded-full bg-blue-300 text-white px-2" @click="addNewWord">Add new word</button></div>


      <input class="h-8 p-2 ml-4" v-model="addNewInput" />
    </div>
  </div>
  <div >
    <div>
      <div class="grid grid-cols-2">
        <ul v-if="hasWords" class="draggable-list">
          <draggable v-model="words" item-key="word" @change="(val) => updateLocalStorage(val)">
            <template #item="{ element }">
              <li :key="element">
                <WordItem :value="element" :remove="() => removeWord(element)" />
              </li>
            </template>
          </draggable>
        </ul>
        <div v-else>Loading</div>
      </div>
    </div>


  </div>

</template>

<style scoped>
.draggable-list {
  list-style: none;
  padding: 0;
  margin: 0;
}
</style>
