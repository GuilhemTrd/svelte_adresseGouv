<script>
    import { onMount } from 'svelte';
    import { debounce } from 'lodash-es';

    let address = '';
    let addresses = [];
    let myAdresses = [];

    let checkAdd = false;
    let checkClick = false;

    const search = debounce(async () => {
        if (address.length < 3) {
            addresses = [];
            return;
        }
        const response = await fetch(`https://api-adresse.data.gouv.fr/search/?q=${address}&limit=5`);
        const data = await response.json();
        addresses = data.features.map(feature => feature.properties.label);
    }, 500);

    const getAdresses = async () => {
        const response = await fetch('http://localhost:1337/api/adresses');
        const data = await response.json();
        myAdresses = data.data.map(adresse => adresse.attributes.name);
        console.log(myAdresses);
    }

    onMount(() => {
        search();
        getAdresses();
    });

    const send = async () => {
        const response = await fetch('http://localhost:1337/api/adresses', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify( {
                data: {
                    name: address
                }
            }),
        });
        if (response.ok) {
            checkAdd = true;
        } else {
            alert('Une erreur s\'est produite lors de l\'envoi de l\'adresse.' + response.statusText);
        }
    }

    const selectAddress = (selectedAddress) => {
        address = selectedAddress;
        addresses = [];
        checkClick = true;
    }
</script>

<div class="container">
    <h1>Adresse</h1>
    {#if checkAdd === true}
        <ul class="address-list">
            <li class="address-item address-item-check"> Ajout effectué !</li>
        </ul>
    {/if}
    <div class="input-group">
        <input type="text" bind:value={address} on:input="{search}" placeholder="Entrer une adresse..." />
        <button class="btn-send" on:click={send}>Envoyer</button>
    </div>
    <div class="card">
        <ul class="address-list">
            <h2> Recherche d'adresses </h2>
            {#each addresses as addressItem (addressItem)}
                <li class="address-item" on:click={() => selectAddress(addressItem)}>{addressItem}</li>
            {/each}
            {#if addresses.length === 0 && address.length > 4 && checkClick === true}
                <li class="address-item address-item-alert">L'adresse n'existe pas</li>
            {/if}
            {#if address.length <= 4 && address.length > 0}
                <li class="address-item address-item-alert"> Votre recherche n'est pas assez précise </li>
            {/if}
            {#if address.length === 0}
                <li class="address-item address-item-alert"> Merci de faire une recherche </li>
            {/if}
        </ul>
    </div>
    <div class="card cardMyAdresse">
        <h2> Mes adresses enregistrées </h2>
        <ul class="address-list">
            {#each myAdresses as myAddress (myAddress)}
                <li class="address-item">{myAddress}</li>
            {/each}
        </ul>
    </div>
</div>

<style>
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        font-family: 'Arial', sans-serif;
        color: #333;
        padding: 20px;
    }

    h1 {
        font-size: 2.5em;
        margin-bottom: 20px;
    }

    .input-group {
        display: flex;
        justify-content: space-between;
        width: 100%;
        margin-bottom: 20px;
    }

    input {
        padding: 10px;
        font-size: 1em;
        border: 1px solid #ccc;
        border-radius: 5px;
        width: 80%;
        box-sizing: border-box;
        transition: border-color 0.3s;
    }

    input:focus {
        border-color: #6699ff;
        outline: none;
    }

    .btn-send {
        padding: 10px 20px;
        background-color: #6699ff;
        border: none;
        border-radius: 5px;
        color: white;
        font-size: 1em;
        cursor: pointer;
        transition: background-color 0.3s;
        width: 18%;
        text-align: center;
    }

    .btn-send:hover {
        background-color: #3366cc;
    }

    .card {
        width: 100%;
        box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        border-radius: 5px;
        padding: 20px;
    }

    .cardMyAdresse {
        margin-top: 20px;
    }

    .address-list {
        list-style: none;
        padding: 0;
    }

    .address-item {
        background-color: #f6f6f6;
        border: 1px solid #ccc;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
        transition: background-color 0.3s;
    }

    .address-item:hover {
        background-color: #e6e6e6;
    }

    .address-item-alert
    {
        background-color: #ffcccc;
        border: 1px solid #ff6666;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
        transition: background-color 0.3s;
    }

    .address-item-alert:hover {
        background-color: #ff9999;
    }

    .address-item-check
    {
        background-color: #ccffcc;
        border: 1px solid #66ff66;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
        transition: background-color 0.3s;
    }

    .address-item-check:hover {
        background-color: #99ff99;
    }

</style>
