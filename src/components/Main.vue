<template>
  <div>
    <h3 v-if="guestNumber >= 0">Total Guests: {{guestNumber}}/20</h3>
    <table>
      <thead>
        <th>Email</th>
        <th>Tickets</th>
        <th></th>
        <th></th>
      </thead>
      <div id="loading_text" v-if="guestNumber < 0">
        <h2>Loading Data...</h2>
      </div>
      <tbody>  
        <tr v-for="(guest, index) in this.guests" :key="index">
          <td>{{guest.email}}</td>
          <td>{{guest.tickets}}</td>
          <td><button :disabled="buttonsDisabled == 1" v-on:click="handleEditClick(index)">Edit</button></td>
          <td><button :disabled="buttonsDisabled == 1" v-on:click="handleDeleteClick(index)">Delete</button></td>
        </tr>      
        <tr id="add_guest_row" v-if="guestNumber < 20">
          <td><input v-model="addValues.email" placeholder="Email"></td>
          <td><input v-model="addValues.tickets" placeholder="Tickets"></td>
          <td><button :disabled="buttonsDisabled == 1" v-on:click="handleAddClick()">Add</button></td>
          <td></td>
        </tr>
      </tbody>
    </table>
    
    <div v-if="editModalHidden == 0" id="edit_modal">
      <div>Edit Guest Information</div>    
      <div> 
        Email 
        <input v-model="editValues.email" placeholder="Email">
      </div>
      <div>
        Tickets
        <input v-model="editValues.tickets" placeholder="Tickets">
      </div> 
      <div>
        <button v-on:click="handleSaveClick()">Save</button>
        <button v-on:click="handleCancelClick()">Cancel</button>
      </div>
    </div>

    <div v-if="editModalHidden == 0" v-on:click="handleCancelClick()" id="page_mask"></div>

  </div>
</template>

<script>
  // eslint-disable-next-line no-unused-vars

  class Guest {
    constructor(){
      this.email = "";
      this.tickets = "";
    }
  }

  const GuestRepository = require('../guest-repository');
  let guestRepo = new GuestRepository();

  export default {
    data: () => {
        return { 
          guests: [], guestNumber: -1, addValues: new Guest(), 
          editValues: new Guest(), editIndex: 0, editModalHidden: 1, 
          buttonsDisabled: 0 
        };
    },
    mounted() {
      this.loadGuests();
    },
    methods: {
        /**
         * Get all guests from the database
         */
        async loadGuests() {
          this.guests = await guestRepo.load();
          this.guestNumber = this.guests.length;
        },
        /**
         * Open guest edit modal and populate input values
         * @param {*} index - guest index to edit
         */
        handleEditClick(index) {
          this.editModalHidden = 0;
          this.editValues.email = this.guests[index].email;
          this.editValues.tickets = this.guests[index].tickets;
          this.editIndex = index;
        },
        /**
         * Update guest from edit modal inputs and close modal
         */
        async handleSaveClick() {
          this.guests[this.editIndex].email = this.editValues.email;
          this.guests[this.editIndex].tickets = this.editValues.tickets;
          this.editModalHidden = 1;
          this.saveToGuestRepo();
        },
        /**
         * Signal to hide modal and modal mask
         */
        handleCancelClick() {
          this.editModalHidden = 1;
        },
        /**
         * Delete guest from guest list
         * @param {int} index - guest index to delete
         */
        async handleDeleteClick(index) {
          this.guests.splice(index, 1);
          this.guestNumber = this.guestNumber - 1;
          this.saveToGuestRepo();
        },
        /**
         * Create new guest entry from add inputs
         */
        async handleAddClick() {
          this.guestNumber = this.guests.push({email: this.addValues.email, tickets: this.addValues.tickets});
          this.addValues.email = "";
          this.addValues.tickets = "";
          this.saveToGuestRepo();
        },
        /**
         * Save guest list to repo; disable edit/save/delete buttons until save completes
         */
        async saveToGuestRepo() {
          this.buttonsDisabled = 1;
          await guestRepo.save(this.guests);
          this.buttonsDisabled = 0;
        },
    },
}
</script>

<style scoped>
  /*Table Styling*/
  td, th {
    overflow: hidden;
    white-space: nowrap;
    display: inline-block;
    width: 25%;
    text-align: left;
    height: 30px;
    padding-left: 5px;
    box-sizing: border-box;
    padding-top: 5px;
  }

  th {
    border-bottom: solid black 1px;
    margin-bottom: 5px;
    margin-top: 5px;
  }

  tr:nth-child(even) {
    background-color: lightgrey;
  }

  tr:nth-child(odd) {
    background-color: white;
  }

  table {
    width: 90%;
    table-layout: fixed;
    border-collapse: collapse;
    margin: auto;
  }

  #add_guest_row {
    background-color: white;
  }

  #add_guest_row button {
    width: 100px;
  }

  /*Button Styling*/
  button {
    border: solid gray 1px;
  }

  button:hover {
    cursor: pointer;
    background-color: lightgrey;
  }

  /*Modal Styling*/
  #edit_modal {
  border: 1px solid black;
  z-index: 1;
  width: 40%;
  height: 40%;
  position: absolute;
  left: 30%;
  top: 10%;
  box-sizing: border-box;
  border-radius: 5px;
  display: inline-block;
  background-color: white;
  }

  #edit_modal div {
    margin: 5px auto;
    text-align: center;
    margin-top: 15px;
    margin-bottom: 15px;
  }

  #edit_modal button {
    margin-left:15px;
    margin-right:15px;
    width: 30%;
    height: 20%;
  }

  #edit_modal input {
    margin-left:15px;
    margin-right:15px;
    width: 70%;
    height: 30%;
  }

  #page_mask {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.7);
  }

  /*Misc. Styling*/
  #loading_text {
    text-align: center;
  }

</style>
