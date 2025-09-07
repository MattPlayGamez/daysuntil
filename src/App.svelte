<script>
    import vacationData from "./assets/vacations.json";

    let today = new Date();
    let todayHTML = today.toISOString().split("T")[0];
    $: today = new Date(todayHTML);

    let customVacations = JSON.parse(localStorage.getItem("customVacations") || "[]");

    function buildVacationList() {
        return [...vacationData, ...customVacations].map(v => ({
            ...v,
            start: new Date(v.start),
            end: new Date(v.end),
        })).sort((a, b) => a.start - b.start);
    }

    let vacationList = buildVacationList();

    function calcFirstSchoolDay(year) {
        let d = new Date(year, 8, 1); // 1 Sept
        if (d.getDay() === 0) d.setDate(2); // Sunday → Monday
        if (d.getDay() === 6) d.setDate(3); // Saturday → Monday
        return d;
    }
    $: eersteSchoolDag = calcFirstSchoolDay(today.getFullYear());

    class Vacation {
        constructor(list, firstSchoolDay) {
            this.firstSchoolDay = firstSchoolDay;
            this.vacationList = list.filter(v => v.end >= today);
        }

        nextVacation() {
            let next = this.vacationList.find(v => v.start >= today);
            if (!next) return null;
            let nogTeDoen = this.daysBetween(today, next.start); // dagen tot vakantie
            let elapsed = this.daysBetween(this.firstSchoolDay, today) + 1; // vandaag ook tellen
            return {
                nextVacation: next,
                nogTeDoen,
                elapsed,
                procent: this.percentage(elapsed, nogTeDoen),
            };
        }

        endOfSchoolYear() {
            let zomer = this.vacationList.find(v => v.name.includes("Zomer"));
            if (!zomer) return null;
            let elapsed = this.daysBetween(this.firstSchoolDay, today) + 1; // vandaag ook tellen
            let nogTeDoen = this.daysBetween(today, zomer.start);
            return {
                elapsed,
                nogTeDoen,
                procent: this.percentage(elapsed, nogTeDoen),
            };
        }

        daysBetween(a, b) {
            return Math.max(0, Math.floor((b - a) / 86400000));
        }
        percentage(a, b) {
            return ((a / (a + b)) * 100).toFixed(2);
        }
    }


    $: manager = new Vacation(vacationList, eersteSchoolDag);
    $: nextVacation = manager.nextVacation();
    $: endYear = manager.endOfSchoolYear();

    // --- Custom vacation handlers ---
    let newName = "";
    let newStart = "";
    let newEnd = "";

    function addCustomVacation() {
        if (!newName || !newStart || !newEnd) return;
        let id = Date.now();
        const newVac = { name: newName, id, start: newStart, end: newEnd, custom: true };
        customVacations = [...customVacations, newVac];
        localStorage.setItem("customVacations", JSON.stringify(customVacations));
        vacationList = buildVacationList();
        newName = newStart = newEnd = "";
    }

    function deleteCustomVacation(id) {
        customVacations = customVacations.filter(v => v.id !== id);
        localStorage.setItem("customVacations", JSON.stringify(customVacations));
        vacationList = buildVacationList();
    }

    function formatDate(d) {
        return (d instanceof Date ? d : new Date(d)).toISOString().split("T")[0];
    }
</script>


<main>
    <h1>🎒 School Aftellen</h1>

<!--    <section class="date-picker">-->
        <h3 class="date-picker" >Vandaag: {todayHTML}</h3>
<!--        <input type="date" bind:value={todayHTML} />-->
<!--    </section>-->

    <section class="cards">
        <div class="card next-vacation">
            <h2>Volgende vakantie</h2>
            <p class="vac-name">{nextVacation.nextVacation.name}</p>
            <p>Resterende dagen: <span>{nextVacation.nogTeDoen}</span></p>
            <p>Verstreken: <span>{nextVacation.elapsed}</span> days</p>
            <div class="progress">
                <div class="bar" style="--progress:{endYear?.procent || 0}%"></div>
            </div>

            <p>Voorgang: {nextVacation.procent}%</p>
        </div>

        <div class="card end-year">
            <h2>Einde van schooljaar</h2>
            <p>Resterende dagen: <span>{manager.endOfSchoolYear().nogTeDoen}</span></p>
            <p>Voortgang: {manager.endOfSchoolYear().procent}%</p>
            <div class="progress">
                <div class="bar" style="width:{manager.endOfSchoolYear().procent}%"></div>
            </div>
        </div>
    </section>

    <section class="add-vacation">
        <h2>Voeg een eigen vakantie toe</h2>
        <div class="form">
            <input placeholder="Naam Vakantie" bind:value={newName} />
            <div class="date-inputs">
                <label>Start:</label>
                <input type="date" bind:value={newStart} />
                <div>
                <label>Einde:</label>
                    <input type="date" bind:value={newEnd} />

                </div>
            </div>
        </div>
        <button on:click={addCustomVacation}>Voeg toe</button>
    </section>

    <section class="upcoming-vacations">
        <h2>Aankomende vakanties</h2>
        <ul>
            {#each vacationList as v}
                <li class="{v.custom ? 'custom' : ''}">
                    <span>{v.name}:</span> {formatDate(v.start)} → {formatDate(v.end)}
                    {#if v.custom}
                        <button class="delete-btn" on:click={() => deleteCustomVacation(v.id)}>Delete</button>
                    {/if}
                </li>
            {/each}
        </ul>
    </section>
</main>

<style>
    /* Mobile-first layout */
    main {
        font-family: 'Segoe UI', sans-serif;
        padding: 1rem;
        background: linear-gradient(to right, #f0f4f8, #d9e2ec);
        color: #333;
    }

    h1 {
        text-align: center;
        font-size: 2rem;
        margin-bottom: 1.5rem;
        color: #1d3557;
    }

    .date-picker {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-bottom: 1.5rem;
    }

    .date-picker input {
        padding: 0.6rem;
        font-size: 1rem;
        border-radius: 1rem;
        border: 2px solid #ccc;
        outline: none;
        transition: border-color 0.2s, box-shadow 0.2s;
    }

    .date-picker input:focus {
        border-color: #1d3557;
        box-shadow: 0 0 10px rgba(29,53,87,0.3);
    }

    .cards {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        margin-bottom: 1.5rem;
    }

    .card {
        background: #fff;
        padding: 1rem;
        border-radius: 1rem;
        box-shadow: 0 8px 15px rgba(0,0,0,0.1);
        transition: transform 0.2s, box-shadow 0.2s;
    }

    .card:hover {
        transform: translateY(-5px);
        box-shadow: 0 12px 25px rgba(0,0,0,0.15);
    }

    .next-vacation { border-left: 6px solid #ff6b6b; }
    .end-year { border-left: 6px solid #4ecdc4; }

    .vac-name {
        font-size: 1.2rem;
        font-weight: bold;
        margin-bottom: 0.5rem;
    }

    .progress {
        height: 12px;
        background: #eee;
        border-radius: 6px;
        overflow: hidden;
        margin: 0.5rem 0;
    }

    .bar {
        height: 100%;
        background: linear-gradient(90deg, #ff6b6b, #ffb347);
        border-radius: 6px;
        width: var(--progress, 0);
        transition: width 0.6s ease;
    }


    @keyframes fill {
        from { width: 0; }
        to { width: var(--width); }
    }

    .add-vacation {
        text-align: center;
        margin-bottom: 1.5rem;
    }

    .add-vacation .form {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
        margin-bottom: 0.5rem;
    }

    .add-vacation .date-inputs {
        display: flex;
        justify-content: center;
        gap: 0.5rem;
        flex-wrap: wrap;
        align-items: center;
    }

    .add-vacation input {
        padding: 0.6rem;
        border-radius: 1rem;
        border: 2px solid #ccc;
        outline: none;
        transition: border-color 0.2s, box-shadow 0.2s;
    }

    .add-vacation input:focus {
        border-color: #1d3557;
        box-shadow: 0 0 8px rgba(29,53,87,0.2);
    }

    .add-vacation button {
        padding: 0.6rem 1rem;
        border: none;
        border-radius: 1rem;
        background: #1d3557;
        color: #fff;
        font-weight: bold;
        cursor: pointer;
        transition: background 0.2s;
    }

    .add-vacation button:hover {
        background: #457b9d;
    }

    .upcoming-vacations ul {
        list-style: none;
        padding: 0;
    }

    .upcoming-vacations li {
        background: #fff;
        margin: 0.3rem 0;
        padding: 0.6rem;
        border-radius: 0.8rem;
        box-shadow: 0 4px 10px rgba(0,0,0,0.08);
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-weight: 500;
        color: #1d3557;
    }

    .upcoming-vacations li.custom {
        border-left: 4px solid #ff6b6b;
        background: #fff3f3;
    }

    .delete-btn {
        background: #ff6b6b;
        border: none;
        color: white;
        padding: 0.3rem 0.6rem;
        border-radius: 0.6rem;
        cursor: pointer;
        transition: background 0.2s;
    }

    .delete-btn:hover {
        background: #ff3b3b;
    }

    @media (min-width: 700px) {
        .cards {
            flex-direction: row;
            justify-content: center;
        }

        .cards .card {
            width: 300px;
        }

        .add-vacation .form {
            flex-direction: row;
            justify-content: center;
        }

        .add-vacation input {
            width: 140px;
        }
    }
</style>
