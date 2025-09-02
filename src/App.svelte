<script>
    import vacationData from "./assets/vacations.json";

    let today = new Date();
    let todayHTML = today.toISOString().split('T')[0];

    // Load custom vacations from localStorage
    let customVacations = JSON.parse(localStorage.getItem('customVacations') || '[]');

    // Combine built-in + custom vacations
    let vacationList = [...vacationData, ...customVacations].map(v => ({
        ...v,
        start: new Date(v.start),
        end: new Date(v.end)
    }));

    const sortVacations = () => {
        vacationList.sort((a, b) => a.start - b.start);
    };

    sortVacations();

    let firstDayOfSeptember = new Date(today.getFullYear(), 8, 1);
    let eersteSchoolDag = firstDayOfSeptember;
    let firstDay = firstDayOfSeptember.getDay();
    if (firstDay === 0) eersteSchoolDag.setDate(firstDayOfSeptember.getDate() + 1);
    if (firstDay === 5) eersteSchoolDag.setDate(firstDayOfSeptember.getDate() + 2);

    class Vacation {
        constructor(listOfDates, firstSchoolDay) {
            this.firstSchoolDay = firstSchoolDay;
            this.vacationList = listOfDates.filter(v => v.start >= today);
        }

        nextVacation() {
            let next = this.vacationList[0];
            let nogTeDoen = this.daysToDo(today, next.start);
            let elapsed = this.elapsedDays(today, this.firstSchoolDay);
            return { nextVacation: next, nogTeDoen, elapsed, procent: this.percentage(elapsed, nogTeDoen) };
        }

        endOfSchoolYear() {
            let zomer = this.vacationList.find(v => v.name.includes("Zomer")).start;
            let elapsed = this.elapsedDays(today, this.firstSchoolDay);
            let nogTeDoen = this.daysToDo(today, zomer);
            return { elapsed, nogTeDoen, procent: this.percentage(elapsed, nogTeDoen) };
        }

        elapsedDays(a, b) { return Math.floor((a - b) / 1000 / 60 / 60 / 24) + 1; }
        daysToDo(a, b) { return Math.floor((b - a) / 1000 / 60 / 60 / 24); }
        percentage(a, b) { return ((a / (a + b)) * 100).toFixed(2); }
    }

    let manager = new Vacation(vacationList, eersteSchoolDag);
    let nextVacation = manager.nextVacation();
    $: today = new Date(todayHTML);
    $: nextVacation = manager.nextVacation();

    // Custom vacation inputs
    let newName = '';
    let newStart = '';
    let newEnd = '';

    function addCustomVacation() {
        if (!newName || !newStart || !newEnd) return
        let id = Math.floor(Math.random()*1000000)
        alert(id)
        const newVac = { name: newName, id: id, start: newStart, end: newEnd, custom: true };
        customVacations.push(newVac);
        localStorage.setItem('customVacations', JSON.stringify(customVacations));
        vacationList = [...vacationData, ...customVacations].map(v => ({
            ...v,
            start: new Date(v.start),
            end: new Date(v.end)
        }));
        sortVacations();
        manager = new Vacation(vacationList, eersteSchoolDag);
        nextVacation = manager.nextVacation();
        newName = newStart = newEnd = '';
    }

    function deleteCustomVacation(id) {
        customVacations = customVacations.filter(v => v.id !== id);
        localStorage.setItem('customVacations', JSON.stringify(customVacations));
        vacationList = [...vacationData, ...customVacations].map(v => ({
            ...v,
            start: new Date(v.start),
            end: new Date(v.end)
        }));
        sortVacations();
        manager = new Vacation(vacationList, eersteSchoolDag);
        nextVacation = manager.nextVacation();
    }

    function formatDate(d) {
        return (d instanceof Date ? d : new Date(d)).toISOString().split('T')[0];
    }
</script>

<main>
    <h1>🎒 School Countdown</h1>

<!--    <section class="date-picker">-->
        <h3 class="date-picker" >Vandaag: {todayHTML}</h3>
<!--        <input type="date" bind:value={todayHTML} />-->
<!--    </section>-->

    <section class="cards">
        <div class="card next-vacation">
            <h2>Next Vacation</h2>
            <p class="vac-name">{nextVacation.nextVacation.name}</p>
            <p>Days remaining: <span>{nextVacation.nogTeDoen}</span></p>
            <p>Elapsed: <span>{nextVacation.elapsed}</span> days</p>
            <div class="progress">
                <div class="bar" style="width:{nextVacation.procent}%"></div>
            </div>
            <p>Progress: {nextVacation.procent}%</p>
        </div>

        <div class="card end-year">
            <h2>End of School Year</h2>
            <p>Days remaining: <span>{manager.endOfSchoolYear().nogTeDoen}</span></p>
            <p>Progress: {manager.endOfSchoolYear().procent}%</p>
            <div class="progress">
                <div class="bar" style="width:{manager.endOfSchoolYear().procent}%"></div>
            </div>
        </div>
    </section>

    <section class="add-vacation">
        <h2>Add Custom Vacation</h2>
        <div class="form">
            <input placeholder="Vacation Name" bind:value={newName} />
            <div class="date-inputs">
                <label>Start:</label>
                <input type="date" bind:value={newStart} />
                <label>End:</label>
                <input type="date" bind:value={newEnd} />
            </div>
        </div>
        <button on:click={addCustomVacation}>Add Vacation</button>
    </section>

    <section class="upcoming-vacations">
        <h2>Upcoming Vacations</h2>
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
        width: 0;
        animation: fill 1s ease forwards;
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
