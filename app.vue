<template>
  <section class="uk-section">
    <div class="uk-container">

      <h1>Management System</h1>

      <!-- <trash2-icon /> -->


      <!-- file upload to firebase storage
      <div>
        <div @dragover.prevent @drop="onAdvancedUpload" @change="onAdvancedUpload"
          style="border: 2px dashed #ccc; padding: 20px;">
          <input type="file" multiple />
          <span class="uk-link">Attach by dragging & dropping, or click to select one or more files.</span>
          <ProgressBar v-if="upload.progress > 0" style="margin-top: 20px;" :value="upload.progress"></ProgressBar>
        </div>
      </div> -->

    </div>
  </section>
</template>



<script>
import 'primevue/resources/themes/aura-light-green/theme.css'
import 'primeicons/primeicons.css'

import ProgressBar from 'primevue/progressbar';
import Toolbar from 'primevue/toolbar';

import '~/assets/css/uikit.grid.css'
import '~/assets/js/firebase.js'

// authentification
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "firebase/auth";
const auth = getAuth();

// import firestore
import { getFirestore, collection, addDoc, deleteDoc, doc, getDocs, setDoc, updateDoc, deleteField } from "firebase/firestore";

// Get a reference to the database service
const db = getFirestore();

// import storage
import { getStorage, ref, uploadBytesResumable, getDownloadURL } from "firebase/storage";
import { parse } from 'vue/compiler-sfc';

// Get a reference to the storage service
const storage = getStorage();

// Add a new document with a generated id.
const addDocument = async (collectionId, data) => {
  const docRef = await addDoc(collection(db, collectionId), data);
  console.log("Document written with ID: ", docRef.id);
}

// remove a document by id
const removeDocument = async (collectionId, id) => {
  await deleteDoc(doc(db, collectionId, id));
  console.log("Document with ID: ", id, " deleted");
}

// list all documents
const listDocuments = async (collectionId) => {
  const querySnapshot = await getDocs(collection(db, collectionId));
  querySnapshot.forEach((doc) => {
    console.log(doc.id);
  });
}

// list all fields of a document
const listFieldsInDocument = async (collectionId) => {
  const querySnapshot = await getDocs(collection(db, collectionId));
  querySnapshot.forEach((doc) => {
    console.log(doc.data());
  });
}

// delete all documents in a collection
const deleteAllDocuments = async (collectionId) => {
  const querySnapshot = await getDocs(collection(db, collectionId));
  querySnapshot.forEach((doc) => {
    removeDocument(collectionId, doc.id);
  });
}

// change the data in a document
const setDocument = async (collectionId, id, data) => {
  const docRef = doc(db, collectionId, id);
  await setDoc(docRef, data);
  console.log("Document with ID: ", id, " updated");
}

// append data to a document
const appendDataToDocument = async (collectionId, id, data) => {
  const docRef = doc(db, collectionId, id);
  await setDoc(docRef, data, { merge: true });
  console.log("Document with ID: ", id, " updated");
}

// delete field in a document
const deleteFieldInDocument = async (collectionId, id, field) => {
  const docRef = doc(db, collectionId, id);
  await updateDoc(docRef, {
    [field]: deleteField()
  });
  console.log("Field ", field, " deleted from document with ID: ", id);
}

export default {
  data() {
    return {
      upload: {
        progress: 0
      }
    }
  },
  methods: {
    onAdvancedUpload(event) {

      console.log(event.target.files[0]);

      // upload files to firebase storage
      const storageRef = ref(storage, 'images/' + event.target.files[0].name);
      const uploadTask = uploadBytesResumable(storageRef, event.target.files[0]);

      uploadTask.on('state_changed',
        (snapshot) => {
          // Observe state change events such as progress, pause, and resume
          // Get task progress, including the number of bytes uploaded and the total number of bytes to be uploaded
          const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
          this.upload.progress = parseInt(progress);
          console.log('Upload is ' + parseInt(progress) + '% done');
        },
        (error) => {
          // Handle unsuccessful uploads
          console.log(error);
        },
        () => {
          // Handle successful uploads on complete
          getDownloadURL(uploadTask.snapshot.ref).then((downloadURL) => {
            console.log('File available at', downloadURL);

            // wait 3seconds than remove the progress bar
            setTimeout(() => {
              this.upload.progress = 0;
            }, 1500);
          });
        }
      );


    }
  },
  mounted() {

    // addDocument("company", {
    //   name: "Tokyo",
    //   country: "Japan"
    // });

    // setDocument("company", "zJhn8x3u5p9tGWhXrl1L", {
    //   name: "Berlin",
    //   country: "Germany"
    // });

    // appendDataToDocument("company", "zJhn8x3u5p9tGWhXrl1L", {
    //   population: 10000000
    // });

    // deleteFieldInDocument("company", "zJhn8x3u5p9tGWhXrl1L", "population");

    // removeDocument("company", "MysCQvOZQUsUepbXfA1N");

    // listDocuments("company");

    // listFieldsInDocument("company");

    // deleteAllDocuments("company");

  }
}

</script>

