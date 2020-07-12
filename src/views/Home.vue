<template>
<div class="wrapper">
    <div class="home">
    <h2>Chat App</h2>
    <h5>Refres Browser to see new data added</h5>
    <hr>
    <br>

    <div class="row">
  <div class="col-1 pack">
       <h4>Register</h4>
    <form action="#">
      <p style="color:green;font-size:11px;" v-show="userform">User Added Succesfully</p>
     <input  type="text" placeholder="Name" v-model="user.name">
     <br><br>
      <input type="text" placeholder="Phone Number" v-model="user.phone">
      <br><br>
      <button @click="register" class="button">Add User</button>
    </form>
    </div>

      <div class="col-2 pack">
       <h4>All User</h4>
    <div v-for="user in allusers" :key="user.id" class="user-box">
    {{ user.id }} | {{ user.name }} | {{ user.phone }}
    </div>
    </div>
    </div>

        <div class="row">
  <div class="col-1 pack">
       <h4>Login </h4>
        <p style="color:green;font-size:11px;" v-show="loginform">Login Successfully</p>
    <form action="#">
     <input  type="text" placeholder="Your Id" v-model="loginuser.id">
     <br><br>
      <button @click="loginAndGetUsers" class="button">Login & fetch Users</button>
    </form>
    <h4>Add a Friend</h4>
          <p style="color:green;font-size:11px;" v-show="adduserform">Your friend has been addded</p>
        <form action="#">
          <input  type="text" placeholder="Adder Id" v-model="adderuser.id">
     <br><br>
     <input  type="text" placeholder="Friend id" v-model="frienduser.id">
     <br><br>
      <button @click="addAFriend" class="button">Add Friend</button>
    </form>
    <br>
    </div>

           <div class="col-2 pack">
            <h4 v-if="userDetails.name != null"> {{userDetails.name}}({{usersFriends.length}})</h4>
       <h4 v-else>Login to show your friends</h4>
       <div v-if="usersFriends.length > 0">
         <div v-for="user in usersFriends" :key="user.id" class="user-box">
      {{ user.id }} | {{ user.name }} | {{ user.phone }}
    </div>  
       </div>
    <div v-else class="user-box">
        You dont have any Contact. Add a Friend 
       </div>
    </div>
    </div>
  
   
  
  </div>
</div>

</template>

<script>
import db from './firebaseinit.js';
import firebase from 'firebase';
export default {
  name: 'Home',
  data(){
    return{
      user: {
        id: '',
        name: "",
        phone: ""
      },

      loginuser: {
        id: '',
        name: "",
        phone: ""
      },
        adderuser:{
           id: ''
        },
      frienduser: {
        id: ''
      },
      userform: false,
      loginform: false,
      adduserform: false,

      allusers:[],

      userDetails: {
         id: null,
        name: null,
        phone: null
      },

        adderresult: {
         id: null,
        name: null,
        phone: null
      },
      
       addfriendresult: {
         id: null,
        name: null,
        phone: null
      },

      usersFriends:[]

    }
  },
  created(){

    //loads all users on page load
                   db.collection('users').get().then(
        querySnapshot => {
          querySnapshot.forEach(doc =>{
           const data  = {
                 'id' : doc.data().UserId,
                 'name' : doc.data().Fullname,
              'phone' : doc.data().PhoneNumber,
           }
           this.allusers.push(data)
          })
        } 
      )
  },
  methods:{
      //generate random Id for users
      generateUserId(){
            var text = "";
     var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
   
     for (var i = 0; i < 8; i++)
       text += possible.charAt(Math.floor(Math.random() * possible.length));
       //save userid as generated id
       this.user.id = text;
        },


         //user registration method
        register(e){
            e.preventDefault();
            //calls the generate id method
           this.generateUserId()
           //save the new user in db
            db.collection('users').doc(this.user.id).set({
                UserId: this.user.id,
                Fullname: this.user.name,
                PhoneNumber: this.user.phone
             }).then(()=>{ this.userform = true }).catch(error => alert(error)) 
        },

        loginAndGetUsers(e){
             e.preventDefault();
             //clears the friends array
             this.usersFriends = [];

             //looks for the user exists
           db.collection('users').doc(this.loginuser.id).get().then(
       (doc) => {
                     if (doc.exists) {
          this.userDetails.id = doc.data().UserId
            this.userDetails.name = doc.data().Fullname
            this.userDetails.phone = doc.data().PhoneNumber

    } else {
        // doc.data() will be undefined in this case
        console.log("No such document!");
    }
          }).then(()=>{
            //if the user exists he loads the contactlist collection
                   db.collection('users').doc(this.loginuser.id).collection('contactList').get().then(
        querySnapshot => {
          querySnapshot.forEach(doc =>{
           const data  = {
                 'id' : doc.data().UserId,
                 'name' : doc.data().Fullname,
              'phone' : doc.data().PhoneNumber,
           }
           this.usersFriends.push(data)
          })
        } 
      )
      }).then(()=>{ this.loginform = true }).catch(error => alert(error)) 
      
        },
        // Add a friend to contactlist
       async addAFriend(e){
          e.preventDefault();

           //checks if the friend is a member
          const isFriendAUser = await db.collection('users').doc(this.frienduser.id).get()

            if(isFriendAUser.exists){
              //if friend is a member, checks if he is already a friend
        const getUser = await db.collection('users').doc(this.adderuser.id).collection('contactList').doc(this.frienduser.id).get()
  if( getUser.exists){
    //if friend is already a friend alert is called
       alert("User Exists as your friend already")
  }
  else{
    //if friend is not a friend, gets the friends details
           db.collection('users').where('UserId', '==', this.frienduser.id).get().then(
            querySnapshot => {
              querySnapshot.forEach(doc =>{
                    this.addfriendresult.id = doc.data().UserId
                this.addfriendresult.name = doc.data().Fullname
                this.addfriendresult.phone = doc.data().PhoneNumber
              })
            }
          ).then(()=>{

                 //gets the person adding details also
                  db.collection('users').where('UserId', '==', this.adderuser.id).get().then(
            querySnapshot => {
              querySnapshot.forEach(doc =>{
                    this.adderresult.id = doc.data().UserId
                this.adderresult.name = doc.data().Fullname
                this.adderresult.phone = doc.data().PhoneNumber
              })
            }
          ).then(()=>{
           //saves the friends details in the contactlist of the person adding
              db.collection('users').doc(this.adderuser.id).collection('contactList').doc(this.addfriendresult.id).set({
                     UserId: this.addfriendresult.id,
                    Fullname: this.addfriendresult.name,
                    PhoneNumber: this.addfriendresult.phone
             })
          }).then(()=>{
             //saves the person adding details also in the friends contactlist
             db.collection('users').doc(this.frienduser.id).collection('contactList').doc(this.adderresult.id).set({
                     UserId: this.adderresult.id,
                    Fullname: this.adderresult.name,
                    PhoneNumber: this.adderresult.phone
             })
          })
          }).then(()=>{ this.adduserform = true }).catch(error => console.log(error)) 
      alert("New Friend Added")
  }
         }
  else{
   alert("This User does not in the system")
  }
        }


        }
  
}
</script>

<style scoped>
.user-box{
      border: 1px solid #f0f0f0;
  border-radius: 3px;
  padding: 5px;
  font-size:12px;
  margin-bottom: 10px;
}
.row{
  margin: 0 auto;
  width: 100%;
  margin-bottom: 30px;
}
    .row::after {
      content: "";
    display: table;
    clear: both;  
    } 
    .col-1{
        float: left;
        width:45%;
        margin-right: 2%;
        margin-bottom: 20px;
    }
    .col-2{
        float: left;
        width:45%;
        margin-bottom: 20px;
    }
.pack{
    border: 1px solid #f0f0f0;
  border-radius: 10px;
  padding: 10px;
}
input{ 
  height: 30px;
  padding-left: 10px;
}
button{
  background: rgb(27, 95, 231);
  color: white;
  padding: 10px;
}
.wrapper{
  padding: 50px 100px 0 100px;
}
.home{
  border: 1px solid #f0f0f0;
  border-radius: 10px;
  padding: 30px;
}
</style>