<script setup>
import { reactive, ref, onMounted } from 'vue';

const value = ref('');
const from = ref('');
const to = ref('');
const newuseradd = ref('')

const history = ref([])
const user = reactive([]);

const selectimage = ref(null)
function handleimageupload(event) {
    const file = event.target.files[0]
    if (file) {
        const reader = new FileReader()
        reader.onload = (e) => {
            selectimage.value = e.target.result
        }
        reader.readAsDataURL(file)
    }
}

function loaddatafromlocalstore() {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
        const parsedUser = JSON.parse(storedUser);
        user.length = 0;
        user.push(...parsedUser);
    }

    const storedHistory = localStorage.getItem('history');
    if (storedHistory) {
        history.value = JSON.parse(storedHistory);
    }
}

function savedatatolocalstore() {
    localStorage.setItem('user', JSON.stringify(user))
    localStorage.setItem('history', JSON.stringify(history.value))
}

onMounted(() => {
    loaddatafromlocalstore()
})


function useradd() {
    if (newuseradd.value.trim() !== '') {
        user.push({
            id: user.length + 1,
            name: newuseradd.value,
            amount: 100,
            time: new Date().toLocaleTimeString(),
            image: selectimage.value || ''
        })
        newuseradd.value = ''
        selectimage.value = null
        savedatatolocalstore()
    }

}

function removeuser(userName) {
    const userIndex = user.findIndex(item => item.name === userName);
    if (userIndex >= 0) {
        user.splice(userIndex, 1);
        savedatatolocalstore();
    }
}

function calculate() {
    const userfrom = user.find(item => item.name === from.value)
    const userto = user.find(item => item.name === to.value)
    if (userfrom && userto) {
        const amount = parseFloat(value.value)
        if (!isNaN(amount)) {
            if (userfrom.amount >= amount) {
                userfrom.amount -= amount
                userto.amount += amount

                if (userfrom === userto) {
                    userto.value = false
                    alert("your amount is not send")
                } else {
                    history.value.reverse().push({
                        from: userfrom.name,
                        to: userto.name,
                        amount: amount,
                        time: new Date().toLocaleTimeString(),
                    })
                    savedatatolocalstore()
                }

            } else {
                userfrom.value = false
                alert('your amount is empty')
            }
        }
    }
}



</script>

<template>
    <section class="section">
        <div class="main">
            <h1 class="userha">User add</h1>
            <form @submit.prevent="useradd">
                <div class="useradd">
                    <input type="text" v-model="newuseradd" required>

                    <label>
                        <input type="file" @change="handleimageupload" accept="image/*">
                    </label>
                </div>
                <button class="userbtn">Add</button>
            </form>
            <form @submit.prevent="calculate" class="form">

                <label class="value">
                    Value <input type="number" v-model="value">
                </label>
                <h1>From</h1>
                <div class="userlist">
                    <label :for="list.id" v-for="list in user" :key="list.id" class="userbox">
                        <span @click="removeuser(list.name)" class="userremove">x</span>
                        <div class="image">
                            <img :src="list.image" alt="image">
                        </div>
                        <div class="useritem">
                            <input type="radio" :value="list.name" :id="list.id" v-model="from" class="">
                            <p :for="list.id" class=""> {{ list.name }} </p>
                        </div>
                    </label>
                </div>

                <h1>To</h1>
                <div class="userlist">
                    <label :for="'to-' + list.id" v-for="list in user" :key="list.id" class="userbox">
                        <span @click="removeuser(list.name)" class="userremove">x</span>
                        <div class="image">
                            <img :src="list.image" alt="image">
                        </div>
                        <div class="useritem">
                            <input type="radio" :value="list.name" :id="'to-' + list.id" v-model="to" class="">
                            <p :for="'to-' + list.id" class=""> {{ list.name }} </p>
                        </div>
                    </label>
                </div>

                <button type="submit" class="subbtn">Submit</button>
            </form>

            <ul>
                <li v-for="list in user" :key="list.id" class="username">
                    <h1 class="">{{ list.name }}</h1>
                    <h1 class="">{{ list.amount }}$</h1>
                </li>
            </ul>

            <h1 class="historyhed">History</h1>
            <ul>
                <li v-for="(transaction, index) in history" :key="index" class="historylist"
                    :class="{ 'border-b-2 border-black': (index + 1) % 4 === 0 }">
                    <p class="history">
                        {{ transaction.from }} transferred ${{ transaction.amount }} to {{ transaction.to }} at {{
                            transaction.time }}
                    </p>
                </li>
            </ul>
        </div>
    </section>
</template>

<style lang="scss">
:root {

    --text-color: #333;
    --primary-color: #34818b;
    --border-color: #999;
    --text-color: #333;
    --box-shado: 0px 2px 6px 0px gray;
}

.section {
    .main {
        background-color: #f1f1f1;
        max-width: 600px;
        margin: auto;
        margin-top: 5%;
        box-shadow: var(--box-shado);
        padding: 10px 20px;

        .userha {
            text-align: center;
            font-size: 30px;
            color: var(--text-color);
            margin-bottom: 20px;
        }

        form {

            .useradd {
                display: flex;
                justify-content: space-between;
                align-items: center;
                gap: 20px;

                input[type=text] {
                    padding: 10px 20px;
                    width: 100%;
                    max-width: 50%;
                    color: var(--text-color);
                    border: 1px solid var(--border-color);
                    outline: 0;
                    border-radius: 5px;
                    background-color: #fff;
                    font-size: 18px;
                }

                input[type=file] {
                    width: 100%;
                    max-width: 40%;
                    color: var(--text-color);
                }

            }

            .userbtn {
                padding: 8px 20px;
                font-size: 18px;
                width: 100%;
                margin-top: 10px;
                cursor: pointer;
                background-color: var(--primary-color);
                border: 0;
                outline: 0;
                border-radius: 5px;
                color: #fff;
            }

        }

        .form {
            .value {
                display: inline-block;
                margin-top: 20px;

                input {
                    padding: 8px;
                    font-size: 18px;
                    outline: 0;
                    border: 1px solid var(--border-color);
                    border-radius: 5px;
                }
            }

            h1 {
                margin-top: 20px;
                font-size: 25px;
            }

            .userlist {
                display: grid;
                grid-template-columns: repeat(4, 1fr);
                gap: 10px;

                .userbox {
                    display: flex;
                    flex-direction: column;
                    width: 100%;
                    box-shadow: var(--box-shado);
                    position: relative;
                    padding: 10px;
                    border-radius: 8px;
                    margin-top: 10px;

                    .userremove {
                        position: absolute;
                        top: 5px;
                        right: 10px;
                        padding: 5px;
                        cursor: pointer;
                    }

                    .image {
                        width: 40px;
                        height: 40px;
                        margin: auto;
                        position: relative;

                        img {
                            width: 100%;
                            height: 100%;
                            object-fit: cover;
                            border-radius: 50%;
                            border: 1px solid var(--border-color);
                            margin-bottom: 2px;
                            font-size: 12px;
                            text-align: center;
                            top: 0;
                            margin: auto;
                        }
                    }

                    .useritem {
                        display: flex;
                        gap: 5px;

                    }
                }
            }

            .subbtn {
                width: 100%;
                background-color: var(--primary-color);
                padding: 8px 20px;
                font-size: 18px;
                color: #fff;
                margin-top: 20px;
                border-radius: 6px;
                cursor: pointer;
                border: 0;
            }
        }

        ul {
            margin-top: 20px;

            .username {
                display: flex;
                justify-content: space-between;
                padding: 10px;
                box-shadow: var(--box-shado);
                border-radius: 6px;
                margin-top: 5px;

                h1 {
                    font-size: 30px;
                    color: var(--text-color);
                    margin: 0;
                }
            }
        }

        .historyhed {
            margin-top: 20px;
            font-size: 30px;
            text-align: center;
            border-bottom: 2px solid var(--border-color);
            padding: 5px 0;
        }

        ul {
            .historylist {
                list-style: none;

                .history {
                    padding: 10px;
                    margin-top: 4px;
                    margin-bottom: 4px;
                    border-radius: 6px;
                    box-shadow: var(--box-shado);
                    font-size: 20px;
                }
            }
        }

    }
}

@media screen and (max-width: 768px) {
    .section {
        .main {
            margin-top: 0;

            form {
                .useradd {
                    flex-direction: column;
                    justify-content: start;
                    align-items: flex-start;
                    gap: 0;

                    input[type=text] {
                        max-width: 100%;
                        padding: 8px 10px;
                    }

                    input[type=file] {
                        margin-top: 15px;
                        max-width: 100%;
                    }

                }

                .userbtn {
                    font-size: 15px;
                }
            }

            .form {
                .value {
                    font-size: 15px;
                    display: block;

                    input {
                        padding: 6px 12px;
                        width: 100%;
                        margin-top: 5px;
                    }
                }

                h1 {
                    font-size: 18px;
                }

                .userlist {
                    grid-template-columns: repeat(2, 1fr);

                    .userbox {

                        .image {
                            width: 30px;
                            height: 30px;
                            margin-bottom: 3px;
                        }

                        .useritem {
                            font-size: 15px;
                        }
                    }
                }

                .subbtn {
                    font-size: 15px;
                }
            }


            ul {
                .username {
                    h1 {
                        font-size: 18px;
                    }
                }
            }

            .historyhed {
                font-size: 18px;
            }

            ul {
                .historylist {

                    .history {
                        font-size: 14px;
                    }
                }
            }
        }
    }
}</style>
