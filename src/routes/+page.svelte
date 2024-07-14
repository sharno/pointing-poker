<script lang="ts">
    let roomId = "";
    let username = "";
</script>

<main>
    <h1>Pointing Poker</h1>
    <div>
        <input bind:value={roomId} placeholder="Room ID" />
        <input bind:value={username} placeholder="Your Name" />
        <!-- <button on:click={joinRoom}>Join Room</button> -->
    </div>
</main>
