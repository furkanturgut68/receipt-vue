<template>
  <div>
    <div v-if="isLoading" style="align-items: center; justify-content: center; display: flex;">
      <TfSpinner />
    </div>
    <detail-component v-else :detailList="detailList" @deleteClick="deleteItem" @updateClick="updateItem"
      @paymentEmit="handleChildPayMethod" @dateEmit="handleChildDate" @priceEmit="handleChildPrice">
    </detail-component>
  </div>
</template>

<script>
import { useRoute, useRouter } from "vue-router";
import {
  doc,
  query,
  where,
  updateDoc,
  getDocs,
  deleteDoc,
  collection,
  getFirestore,
} from "firebase/firestore";
import { onMounted, ref } from "vue";
import DetailComponent from "@/components/DetailComponent.vue";
export default {
  components: {
    DetailComponent,
  },
  setup() {
    const db = getFirestore();
    const route = useRoute();
    const params = route.params;
    const router = useRouter();
    const detailList = ref([]);
    const upDate = ref(null);
    const updatePayMethod = ref(null);
    const updatePrice = ref(null);
    const isLoading = ref(true);

    onMounted(async () => {
      try {
        const q = query(
          collection(db, "infos"),
          where("slipsId", "==", Number(params.id))
        );

        const querySnapshot = await getDocs(q);
        querySnapshot.forEach((doc) => {
          detailList.value.push(doc.data());
          isLoading.value = false
        });
      } catch (error) {
        console.error("infos database error:", error);
      }
    });

    const updateItem = async (myID) => {
      const dB = getFirestore();
      const q = query(collection(dB, "infos"), where("slipsId", "==", myID));

      const querySnapshot = await getDocs(q);
      if (querySnapshot.size > 0) {
        const docum = querySnapshot.docs[0];
        const firstUpdate = {
          paymentMethod: updatePayMethod.value,
          receiptDate: upDate.value,
          price: Number(updatePrice.value)
        };
        isLoading.value = false

        await updateDoc(docum.ref, firstUpdate);
      }
    };

    const handleChildPayMethod = (newMethod) => {
      updatePayMethod.value = newMethod?.value;
  
    };

    const handleChildPrice = (newPrice) => {
      updatePrice.value = newPrice.value;
    };

    const handleChildDate = (newDate) => {
      upDate.value = newDate.value;
    };

    const deleteItem = (myID) => {
      const q = query(collection(db, "infos"), where("slipsId", "==", myID));
      getDocs(q).then((querySnapshot) => {
        querySnapshot.forEach((docs) => {
          const docRef = doc(db, "infos", docs.id);
          deleteDoc(docRef).then(() => {
            router.push({ name: "HomeView" });
          });
        });
      });
    };

    return {
      detailList,
      deleteItem,
      updateItem,
      handleChildPrice,
      handleChildPayMethod,
      handleChildDate,
      isLoading
    };
  },
};
</script>

<style scoped></style>
