<script>
    import { onMount } from "svelte";
    import { items } from "../store";
    import {v4 as uuidv4} from "uuid";
    import TodoApi from "../TodoApi";
    import Item from "./Item.svelte";
    import NewItem from "./NewItem.svelte";

    function handleNewItem(e){
        $items = [{
            id : uuidv4(),
            text : e.detail,
            completed : false
        }, ...$items]

        TodoApi.save($items);
    }

    function handleUpdate(e){
        const index = $items.findIndex(item => item.id === e.detail.id)
        // to get the index of item from the array of items.
        // console.log($items[index]); // the one inside the store --- wont change with the change in input 
        // console.log(e.detail); // change with the change of input
        $items[index] = e.detail
        // update the store
        TodoApi.save($items) 
        //save the item to the actual API of the local storage ?
    }

    function handleDelete(e){
        $items = $items.filter(item => item.id != e.detail)
        console.log($items);
        TodoApi.save($items); 
    }

    let itemsSorted = []

    $ : {
         itemsSorted = [...$items]
         itemsSorted.sort((a,b) => {
            if (a.completed && b.completed) return 0;
            if (a.completed) return 1;
            if (b.completed) return -1;
         })
    }


    onMount( async () => {
        //fetch from API
        $items = await TodoApi.getAll();
    });

    


</script>

<style>
    .list {
        padding: 15px;
    }

    .list-status {
        margin: 0;
        text-align: center;
        color: #ffffff;
        font-weight: bold;
        font-size: 1.1em;
    }
</style>

<div class="list">
    <NewItem on:newItem={handleNewItem}/>
    {#each itemsSorted as item}
        <Item {...item} on:update={handleUpdate} on:delete={handleDelete} />
    {:else}
        <div class="list-status">No Item Exist</div>
    {/each}
</div>

