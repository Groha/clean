<script>
export default {
  data() {
    return {
      url: 'https://64de108c825d19d9bfb1fa08.mockapi.io/schedule',
      schedule: [],
      filteredSchedule: [],
      showModal: false,
      clickedTodo: null,
      editedTodo: null,
      currentDate: '',
      currentRoom: '',
      currentFloor: '',
      currentName: '',
      modalDate: '',
      modalRoom: '',
      modalFloor: '',
      modalName: '',
      filterRoom: '',
    };
  },
  async mounted() {
    this.schedule = await this.fetchSchedule();
    this.sortByDate();
    this.filteredSchedule = [...this.schedule];
  },
  methods: {
    async fetchSchedule() {
      try {
        const response = await fetch(this.url);
        if (!response.ok) {
          throw new Error('Ошибка при получении расписания');
        }
        return response.json();
      } catch (error) {
        console.error('Ошибка при получении расписания:', error);
        return [];
      }
    },
    openModal(todo) {
      this.showModal = true;
      this.clickedTodo = todo;
      this.editedTodo = { ...todo };
      this.modalDate = todo.date;
      this.modalRoom = todo.room;
      this.modalFloor = todo.floor;
      this.modalName = todo.name;
    },
    async refreshSchedule() {
      this.schedule = await this.fetchSchedule();
    },
    async addRoom() {
      try {
        const newRoom = {
          date: this.currentDate,
          room: this.currentRoom,
          floor: this.currentFloor,
          name: this.currentName
        };
        const response = await fetch(this.url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(newRoom)
        });
        if (!response.ok) {
          throw new Error('Ошибка при добавлении комнаты');
        }
        const addedRoom = await response.json();
        this.schedule.push(addedRoom);

        this.filterRoom = '';

        this.currentDate = '';
        this.currentRoom = '';
        this.currentFloor = '';
        this.currentName = '';

        this.refreshSchedule();

      } catch (error) {
        console.error('Ошибка при добавлении комнаты:', error);
      }
    },
    async updateRoom() {
      if (!this.editedTodo) {
        return;
      }
      try {
        this.editedTodo.date = this.modalDate;
        this.editedTodo.room = this.modalRoom;
        this.editedTodo.floor = this.modalFloor;
        this.editedTodo.name = this.modalName;
        const response = await fetch(`${this.url}/${this.editedTodo.id}`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(this.editedTodo)
        });
        if (!response.ok) {
          throw new Error('Ошибка при обновлении комнаты');
        }
        const indexToUpdate = this.schedule.findIndex(todo => todo.id === this.editedTodo.id);
        if (indexToUpdate !== -1) {
          this.schedule[indexToUpdate] = { ...this.editedTodo };
        }

        this.filterRoom = '';

        this.refreshSchedule();

        this.showModal = false;
      } catch (error) {
        console.error('Ошибка при обновлении комнаты:', error);
      }
    },
    applyFilters() {
      this.filteredSchedule = this.schedule.filter(todo => {
        const roomMatches = !this.filterRoom || todo.room == this.filterRoom;
        return roomMatches;
      });

      this.sortByDate();
    },
    checkRoom() {
      if(this.filterRoom === '') {
        this.filterRoom = '';
        this.refreshSchedule();
        this.applyFilters();
      }
    },
    sortByDate() {
      if (this.sortOrder === 'asc') {
        this.filteredSchedule.sort((a, b) => new Date(a.date) - new Date(b.date));
        this.sortOrder = 'desc';
      } else {
        this.filteredSchedule.sort((a, b) => new Date(b.date) - new Date(a.date));
        this.sortOrder = 'asc';
      }
    },
  },
  computed: {
    uniqueRooms() {
      return [...new Set(this.schedule.map(todo => todo.room))];
    }
  },
  watch: {
    filterRoom: 'applyFilters',
  }
}
</script>


<template>
  <div class="px-2 py-12 md:p-8 mx-auto md:max-w-[80%] overflow-hidden">
    <div class="border border-solid border-gray-300">
      <h1 class="p-5 text-4xl font-bold">График уборки</h1>
      <div class="grid md:grid-cols-5">
        <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="date" placeholder="Дата..." v-model="currentDate">
        <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="number" min="1" max="30" placeholder="№ к-ты..." v-model="currentRoom" />
        <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="number" min="1" max="3" placeholder="Этаж..." v-model="currentFloor" />
        <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="text" placeholder="Имя..." v-model="currentName" />
        <button @click="addRoom()" class="p-3 border border-gray-300 rounded-sm cursor-pointer bg-white transition-all duration-300 m-2 hover:bg-gray-300">Добавить комнату</button>
      </div>
    </div>
    <div class="flex md:flex-col md:max-h-[51vh] overflow-hidden">
      <div class="border border-solid border-gray-300 grid md:grid-cols-5 mt-5">
        <span class="border-t md:border-t-0 md:border-l border-solid border-gray-300 first:border-none font-bold p-5 cursor-pointer flex justify-between items-center" @click="sortByDate">
          Дата
          <span v-if="sortOrder === 'asc'">↑</span>
          <span v-else>↓</span>
        </span>
        <select class="border-t md:border-t-0 md:border-l border-solid border-gray-300 first:border-none font-bold p-5 md:p-2 min-h-[65px] cursor-pointer" v-model="filterRoom" @change="checkRoom()">
          <option value="">Все комнаты</option>
          <option v-for="room in uniqueRooms" :key="room" :value="room">{{ room }}</option>
        </select>
        <span class="border-t md:border-t-0 md:border-l border-solid border-gray-300 first:border-none font-bold p-5">Этаж</span>
        <span class="border-t md:border-t-0 md:border-l border-solid border-gray-300 first:border-none font-bold p-5">Имя</span>
        <span class="border-t md:border-t-0 md:border-l border-solid border-gray-300 first:border-none font-bold p-5">Изменить</span>
      </div>
      <div class="flex md:flex-col overflow-x-scroll mt-5 md:mt-0 md:overflow-y-scroll">
        <div v-for="todo in filteredSchedule" :key="todo.id">
          <div class="border border-gray-300 border-t-0 grid md:grid-cols-5 min-w-[140px]">
            <span class="border-t md:border-t-0 md:border-l border-gray-300 p-5 md:first:border-none min-h-[65px]">{{ todo.date }}</span>
            <span class="border-t md:border-t-0 md:border-l border-gray-300 p-5 md:first:border-none min-h-[65px]">{{ todo.room }}</span>
            <span class="border-t md:border-t-0 md:border-l border-gray-300 p-5 md:first:border-none min-h-[65px]">{{ todo.floor }}</span>
            <span class="border-t md:border-t-0 md:border-l border-gray-300 p-5 md:first:border-none min-h-[65px]">{{ todo.name }}</span>
            <p @click="openModal(todo)" class="m-0 flex justify-center p-5 border-t md:border-t-0 md:border-l border-gray-300 cursor-pointer">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path d="M7.127 22.562l-7.127 1.438 1.438-7.128 5.689 5.69zm1.414-1.414l11.228-11.225-5.69-5.692-11.227 11.227 5.689 5.69zm9.768-21.148l-2.816 2.817 5.691 5.691 2.816-2.819-5.691-5.689z"/></svg>
            </p>
            <div v-if="showModal" class="w-full h-full fixed inset-0 bg-slate-600">
              <div class="grid bg-white w-full md:w-[40%] mx-auto top-[50%] translate-y-[-50%] relative p-5 pt-10">
                <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="date" placeholder="Дата..." v-model="modalDate">
                <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="number" min="1" max="30" placeholder="№ к-ты..." v-model="modalRoom" />
                <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="number" min="1" max="3" placeholder="Этаж..." v-model="modalFloor" />
                <input class="border border-gray-300 p-3 m-2 outline-none w-[calc(100%-1rem)] md:w-auto" type="text" placeholder="Имя..." v-model="modalName" />
                <button @click="updateRoom()" class="p-3 border border-gray-300 rounded-sm cursor-pointer bg-white transition-all duration-300 m-2 hover:bg-gray-300">Сохранить данные</button>
                <span @click="showModal = false" class="absolute top-[10px] right-[14px] font-bold cursor-pointer">x</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>