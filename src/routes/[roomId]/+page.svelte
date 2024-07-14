<script lang="ts">
    import { page } from "$app/stores";
    import {
        getFirestore,
        collection,
        addDoc,
        onSnapshot,
        doc,
        deleteDoc,
    } from "firebase/firestore";
    import { app } from "$lib/firebase";

    interface Vote {
        id: string;
        username: string;
        vote: string;
    }

    let roomId = $page.params.roomId;
    let username = $state("");
    let usernameInput = $state("");
    let participants: string[] = $state([]);
    let selectedCard: number | null = $state(null);
    let votes: Vote[] = $state([]);
    let cards = [0, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89];
    let showVotesFlag = $state(false);

    function setUsername() {
        username = usernameInput;
        joinRoom();
    }

    const nums = $derived(votes.map((v) => parseInt(v.vote)).toSorted());
    const average: string = $derived(
        (nums.reduce((acc, b) => acc + b, 0) / votes.length).toFixed(2),
    );
    const median: string = $derived.by(() => {
        const mid = Math.floor(nums.length / 2);
        return ((nums[mid - 1] + nums[mid]) / 2).toFixed(2);
    });
    const mode: number[] = $derived.by(() => {
        const freq = new Map<number, number>();
        for (const n of nums) {
            const count = freq.get(n) ?? 0;
            freq.set(n, count + 1);
        }
        let mode = [];
        let max = 0;
        for (const kv of freq.entries()) {
            if (kv[1] > max) mode = [];
            if (kv[1] >= max) {
                max = kv[1];
                mode.push(kv[0]);
            }
        }
        return mode;
    });

    const db = getFirestore(app);

    async function castVote() {
        console.log(selectedCard);
        if (selectedCard !== null) {
            const oldVote = votes.find((v) => v.username === username);
            if (oldVote) {
                await deleteDoc(doc(db, "rooms", roomId, "votes", oldVote.id));
            }
            const votesRef = collection(db, "rooms", roomId, "votes");
            await addDoc(votesRef, { username, vote: selectedCard });
        }
    }

    async function resetVotes() {
        const deletePromises = votes.map((vote) =>
            deleteDoc(doc(db, "rooms", roomId, "votes", vote.id)),
        );
        showVotesFlag = false;
        await Promise.all(deletePromises);
    }

    async function showVotes() {
        if (votes.every((vote) => vote.vote !== "")) {
            showVotesFlag = true;
        }
    }

    function joinRoom() {
        if (!roomId || !username) {
            return;
        }
        const votesRef = collection(db, "rooms", roomId, "votes");
        const unsubscribe = onSnapshot(votesRef, (snapshot) => {
            votes = snapshot.docs.map(
                (doc) =>
                    ({
                        id: doc.id,
                        ...doc.data(),
                    }) as Vote,
            );
            participants = Array.from(
                new Set(votes.map((vote) => vote.username)),
            );
            // If any vote is changed to empty, hide all votes
            if (votes.some((vote) => vote.vote === "")) showVotesFlag = false;
        });
    }
</script>

<div>
    <h2>Room: {roomId}</h2>
    <div>
        {#if username === ""}
            <form onsubmit={setUsername}>
                <label for="username">username:</label>
                <input id="username" bind:value={usernameInput} />
                <button>Submit</button>
            </form>
        {:else}
            <h2>Username: {username}</h2>
        {/if}
    </div>
    <div>
        {#each cards as card}
            <button
                class:selected={selectedCard === card}
                onclick={() => (selectedCard = card)}
            >
                {card}
            </button>
        {/each}
    </div>
    <button onclick={castVote}>Cast Vote</button>
    <button onclick={resetVotes}>Reset Votes</button>
    <button onclick={showVotes}>Show Votes</button>

    <h3>Votes:</h3>
    <ul>
        {#each participants as participant}
            <li>
                {participant}:
                {#if showVotesFlag}
                    {votes.find((v) => v.username === participant)?.vote ||
                        "Not voted"}
                {:else}{votes.some((v) => v.username === participant)
                        ? "🤫"
                        : "Not voted"}{/if}
            </li>
        {/each}
    </ul>
    {#if showVotesFlag}
        <h3>Average:</h3>
        <div>{average}</div>
        <h3>Median:</h3>
        <div>{median}</div>
        <h3>Mode:</h3>
        <div>{mode}</div>
    {/if}
</div>

<style>
    .selected {
        background-color: #4caf50;
        color: white;
    }
</style>
