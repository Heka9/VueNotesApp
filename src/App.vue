<script setup>
import {ref} from 'vue'

const showModal = ref(false)
const newNote = ref('')
const notes = ref([])
const errorMsg = ref('')
// const showSaveButton = ref(false)     состояние показа кнопки сохранить изменения, пока убрал

/**
 *
 * @REVIEW
 * Make utils or composables from this function
 */
const getRandomColor = () => {
  return 'hsl(' + (Math.random() * 360).toFixed() + ', 100%, 75%)'
}

const addNote = () => {
  if (newNote.value.length < 5) {
    return (errorMsg.value = 'too short, minimum 5 symbols')
  }
  notes.value.push({
    id: new Date().getTime(),
    text: newNote.value,
    date: new Date().toLocaleDateString('en-GB'),
    backgroundColor: getRandomColor(),
  })
  showModal.value = !showModal.value
  newNote.value = ''
  errorMsg.value = ''
}

const deleteCard = (id) => {
  /**
   *
   * @NOTES
   * MAP повертає масив такої ж самої довжини, він для того щоб відозмінювати значення.
   * FILTER для того щоб повернути новий масив, який прошов певні умови.
   */
  notes.value = notes.value.filter((note) => note.id !== id) // почему не работает с map?
}


/**
 *
 * @REVIEW
 * Тут краще працювати з даними а не з DOM
 * Створити окремий компонент  EditParagraph.vue
 * прокинути props
 * додати emits
 * повертати відредагований контент
 */
const editParagraphContent = (e) => {
  const targetParagraph = e.target.parentElement.nextElementSibling
  const saveButton = e.target.nextElementSibling.nextElementSibling

  targetParagraph.setAttribute('contenteditable', 'true')
  targetParagraph.focus()
  saveButton.style.display = 'block'
  // showSaveButton.value = true
}

/**
 *
 * @REVIEW
 * теж саме що і editParagraphContent це буде один комопнент.
 */
const saveParagraphContent = (e, id) => {
  const targetParagraph = e.target.parentElement.nextElementSibling

  notes.value.forEach((note) => {
    if (note.id === id) {
      note.text = targetParagraph.textContent
    }
  })

  /**
   * @REVIEW
   * Таке краще робити через v-bind
   * const contenteditableVar = ref(false)
   * <p :contenteditable='contenteditable'></p>
   * якщо contenteditableVar === true, то в тега <p> зявиться атрибут contenteditable
   */
  targetParagraph.removeAttribute('contenteditable')
  e.target.style.display = 'none'
  // showSaveButton.value = !showSaveButton.value
}

/**
 * @REVIEW
 * Для того щоб працювати з DOM потрібно використовувати ref;
 * const modalTextarea = ref(null);
 * <p ref='modalTextarea'></p>;
 * такоим чином воно біндить цей p і в modalTextarea буде наш елемент дом.
 */
const openModal = () => {
  showModal.value = !showModal.value
  const textarea = document.querySelector('.modal textarea')
  textarea.focus() // какого хера не работает фокус так и не понял
}

const closeModal = (e) => {
  if (!e.target.closest('.modal')) showModal.value = !showModal.value
}

//разобраться с показом кнопки сохранить изменения
</script>
<template>
  <main>
    <div class="container">
      <div v-if="showModal" @click="closeModal" class="overlay">
        <!-- какая разница с v-show, работает с ним и с v-if -->
        <div class="modal">
          <button class="close" @click="showModal = !showModal">X</button>
          <textarea v-model.trim="newNote" name="note" id="note" cols="30" rows="10"
                    placeholder="Write your note"></textarea>
          <p v-if="errorMsg" class="error">{{ errorMsg }}</p>
          <button @click="addNote" class="add">Add Note</button>
        </div>
      </div>
      <header>
        <h1>Notes</h1>
        <button @click="openModal">+</button>
      </header>
      <div class="card-container">
<!--        @REVIEW - булоб корисно винести цей в окремий компонент.-->
        <div v-for="note in notes" :key="note.id" class="card" :style="{ backgroundColor: note.backgroundColor }">
          <div class="navbar">
            <button @click="editParagraphContent" class="edit">Edit</button>
            <button @click="deleteCard(note.id)" class="delete">Delete</button>
            <button @click="(e) => saveParagraphContent(e, note.id)" class="save">Save</button>
            <!-- убрал стейт для показа кнопки v-if="showSaveButton" т.к.показывало для всех карточек и скрывало у всех -->
          </div>
          <p class="main-text">{{ note.text }}</p>
          <p class="date">{{ note.date }}</p>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  width: 100vw;
  height: 100vh;
}

.container {
  max-width: 980px;
  padding: 10px;
  margin: 0 auto;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

h1 {
  font-weight: 700;
  margin-bottom: 25px;
  font-size: 60px;
}

header button {
  border: none;
  padding: 10px;
  width: 50px;
  flex: 0 0 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(130, 63, 194);
  color: #ffffff;
  border-radius: 50%;
  font-size: 25px;
}

.card-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: 1fr;
  gap: 20px;
}

.card {
  height: 225px;
  overflow: auto;
  padding: 15px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  color: #333;
}

.navbar {
  display: flex;
  gap: 10px;
}

.navbar button {
  border: 1px solid transparent;
  padding: 5px 10px;
  cursor: pointer;
  color: #fff;
  border-radius: 5px;
  background-color: #292828;
}

.save {
  display: none;
}

.main-text {
  font-size: 18px;
  line-height: 1.5;
  flex-grow: 1;
}

.date {
  font-size: 14px;
  font-weight: 700;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  width: 750px;
  background-color: #fff;
  border-radius: 10px;
  padding: 60px 30px 30px;
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.add {
  padding: 10px 20px;
  font-size: 20px;
  width: 100%;
  background-color: #3d58a3;
  border: none;
  color: #fff;
  cursor: pointer;
}

.close {
  position: absolute;
  border: none;
  top: 10px;
  right: 10px;
  flex: 0 0 40px;
  width: 40px;
  height: 40px;
  align-self: flex-start;
  color: #000;
  font-size: 30px;
  cursor: pointer;
  background-color: transparent;
}

textarea {
  resize: vertical;
  min-height: 100px;
  max-height: 250px;
  font-size: 20px;
  padding: 10px;
}

.error {
  color: red;
}
</style>
