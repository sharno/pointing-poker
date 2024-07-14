<script lang="ts">
    import { goto } from "$app/navigation";

    function createRoom() {
        goto(`/${Date.now()}`);
    }
</script>

<main>
    <h1>Pointing Poker</h1>
    <button onclick={createRoom}>Create a room</button>
</main>
