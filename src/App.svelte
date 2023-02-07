<script>
  import Calendar from "./Calendar.svelte";
  import { interrogazioni } from "./store";
  import { onMount } from 'svelte';
  var items = [];
  var dayNames = [
    "Lunedi",
    "Martedi",
    "Mercoledi",
    "Giovedi",
    "Venerdi",
    "Sabato",
    "Domenica",
  ];
  let monthNames = [
    "Gennaio",
    "Febbraio",
    "Marzo",
    "Aprile",
    "Maggio",
    "Giugno",
    "Luglio",
    "Agosto",
    "Settembre",
    "Ottobre",
    "Novembre",
    "Dicembre",
  ];
  let headers = [];
  let now = new Date();
  let year = now.getFullYear(); //	this is the month & year displayed
  let month = now.getMonth();
  let eventText = "Seleziona un evento o una data";
  let nuovoTodo, priority, date, anno, mese, giorno;
  var days = [];
  let allitems = [];
  var currentItemDate = new Date();
  var overlapsLimit = 0;
  let description = "";
  let titleToDelete = "";
  let dateToDelete = "";
  let id = 0;
  onMount(() =>{
    load();
    refresh();
  })
  function AddTodo() {
    var ele = document.getElementsByName("priority");
    var sameDate = false;
    for (let i = 0; i < ele.length; i++) {
      if (ele[i].checked) {
        priority = ele[i].value;
        break;
      }
    }
    if(description.length != 0 && nuovoTodo.length != 0 &&               date.length != 0 && priority.length != 0){
      if(nuovoTodo.length > 11){
        nuovoTodo = nuovoTodo.substring(0,10);
      }
      //Formattazione dei dati
      anno = parseInt(date.substring(0, 4));
      mese = parseInt(date.substring(5, 7)) - 1;
      giorno = parseInt(date.substring(8, 10));

      currentItemDate = new Date(anno, mese, giorno);
  
      for (let j of $interrogazioni) {
        if (j.date.getTime() == currentItemDate.getTime()) {
          sameDate = true;
          overlapsLimit++;
        }
      }
      if (overlapsLimit <= 1) {
        interrogazioni.update((oldValue) => {
          oldValue.push({
            title: nuovoTodo,
            date: currentItemDate,
            className: priority,
            isBottom: sameDate,
            description: description,
            id: id
          });
          return oldValue;
        });
        id++;
        overWrite();
        initContent();
      } else
        alert("Impossibile aggiungere la task richiesta. (max task raggiunte)");
      overlapsLimit = 0;
    }
    else
      alert("No input detected");
  }

  function overWrite() {
    for (var i = 0; i < $interrogazioni.length; i++) {
      allitems[i] = {
        title: $interrogazioni[i].title,
        className: $interrogazioni[i].className,
        date: $interrogazioni[i].date,
        len: 1,
        id: $interrogazioni[i].id,
        isBottom: $interrogazioni[i].isBottom,
        description: $interrogazioni[i].description,
      };
    }

  }
  function save(){
    localStorage.setItem('item', JSON.stringify(allitems));
  }
  function load(){
    let test = JSON.parse(localStorage.getItem('item'));
    for(let i of test){
      
      anno = parseInt(i["date"].substring(0, 4));
      mese = parseInt(i["date"].substring(5, 7)) - 1;
      giorno = parseInt(i["date"].substring(8, 10)) + 1;
      
      currentItemDate = new Date(anno, mese, giorno);
      
      interrogazioni.update((oldValue) => {
        oldValue.push({
          title: i["title"],
          date: currentItemDate,
          className: i["className"],
          isBottom: i["isBottom"],
          description: i["description"],
          id: i["id"]
        });
        return oldValue;
      });
    }
    overWrite();
  }
  function getYearItems() {
    var array2 = [];
    var index = 0;
    for (let i of allitems) {
      if (i.date.getFullYear() == year) {
        array2[index] = i;
        index++;
      }
    }
    return array2;
  }

  function getMonthItems() {
    var year = getYearItems();
    var array = [];
    var index = 0;
    for (let i of year) {
      if (i.date.getMonth() == month) {
        array[index] = i;
        index++;
      }
    }
    return array;
  }

  function initMonthItems() {
    items = getMonthItems();
    //This is where you calc the row/col to put each dated item
    for (let i of items) {
      let rc = findRowCol(i.date);
      if (rc == null) {
        i.startCol = i.startRow = 0;
      } else {
        i.startCol = rc.col;
        i.startRow = rc.row;
      }
    }
  }

  $: month, year, initContent();
  // choose what date/day gets displayed in each date box.

  function initContent() {
    headers = dayNames;
    initMonth();
    initMonthItems();
  }

  function initMonth() {
    days = [];
    let monthAbbrev = monthNames[month].slice(0, 3);
    let nextMonthAbbrev = monthNames[(month + 1) % 12].slice(0, 3);
    //	find the last Monday of the previous month
    var firstDay = new Date(year, month, 1).getDay();
    var daysInThisMonth = new Date(year, month + 1, 0).getDate();
    var daysInLastMonth = new Date(year, month, 0).getDate();
    var prevMonth = month == 0 ? 11 : month - 1;

    //	show the days before the start of this month (disabled) - always less than 7
    for (let i = daysInLastMonth - firstDay; i < daysInLastMonth; i++) {
      let d = new Date(prevMonth == 11 ? year - 1 : year, prevMonth, i + 1);
      days.push({ name: "" + (i + 1), enabled: false, date: d });
    }
    //	show the days in this month (enabled) - always 28 - 31
    for (let i = 0; i < daysInThisMonth; i++) {
      let d = new Date(year, month, i + 1);
      if (i == 0)
        days.push({
          name: monthAbbrev + " " + (i + 1),
          enabled: true,
          date: d,
        });
      else days.push({ name: "" + (i + 1), enabled: true, date: d });
    }
    //	show any days to fill up the last row (disabled) - always less than 7
    for (let i = 0; days.length % 7; i++) {
      let d = new Date(month == 11 ? year + 1 : year, (month + 1) % 12, i + 1);
      if (i == 0)
        days.push({
          name: nextMonthAbbrev + " " + (i + 1),
          enabled: false,
          date: d,
        });
      else days.push({ name: "" + (i + 1), enabled: false, date: d });
    }
  }

  function findRowCol(dt) {
    for (let i = 0; i < days.length; i++) {
      let d = days[i].date;
      if (
        d.getYear() === dt.getYear() &&
        d.getMonth() === dt.getMonth() &&
        d.getDate() === dt.getDate()
      )
        return { row: Math.floor(i / 7) + 2, col: (i % 7) + 1 };
    }
    return null;
  }

  function itemClick(e) {
    eventText = "Descrizione: " + e.description;
  }

  function next() {
    month++;
    if (month == 12) {
      year++;
      month = 0;
    }
    initContent();
  }

  function prev() {
    if (month == 0) {
      month = 11;
      year--;
    } else {
      month--;
    }
    initContent();
  }
  
  function refresh() {
    next();
    prev();
  }

  function UpdateStore(id){
    $interrogazioni = $interrogazioni.filter(i => i.id != id );
    allitems = allitems.filter(s => s.id != id)
  }
  function Delete() {
    if(dateToDelete.length != 0 && titleToDelete.length != 0){
      let deleteYear = parseInt(dateToDelete.substring(0, 4));
      let deleteMonth = parseInt(dateToDelete.substring(5, 7)) -             1;
      let deleteDay = parseInt(dateToDelete.substring(8, 10));
      let deleteDate = new Date(deleteYear, deleteMonth,                                 deleteDay);
     
      for (let i of $interrogazioni) {
        if (i.title == titleToDelete &&
          i.date.getTime() == deleteDate.getTime()) {
          UpdateStore(i.id);
            for(let j of $interrogazioni){
              if(j.date.getTime() == deleteDate.getTime()){
                j.isBottom = false;
              }
            }    
          break;
          }
      }
      overWrite();
      refresh();
    }
    else
      alert("No input detected");
  }
</script>

<div class="calendar-container">
  <div class="calendar-header"> 
    <h1>
      <button on:click={() => year--}>&Lt;</button>
      <button on:click={() => prev()}>&lt;</button>
      {monthNames[month]}
      {year}
      <button on:click={() => next()}>&gt;</button>
      <button on:click={() => year++}>&Gt;</button>
    </h1>
    {eventText}
  </div>

  <Calendar
    {headers}
    {days}
    {items}
    on:scroll={refresh()}
    on:itemClick={(e) => itemClick(e.detail)}
  />
</div>
<input type="checkbox" id="addEv" class="modal-toggle" />
<div class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Aggiungi un evento al calendario</h3>
    <br />
    <input
      type="text"
      bind:value={nuovoTodo}
      placeholder="Titolo"
      class="input input-ghost w-full max-w-xs"
    />
    <br /><br />
    <textarea
      class="textarea textarea-ghost"
      placeholder="Descrizione"
      bind:value={description}
    />
    <br /><br />
    <input type="date" bind:value={date} />
    <br /><br />
    <div class="c">
      <input type="radio" name="priority" value="task--danger" /> Alta
      <input type="radio" name="priority" value="task--warning" /> Media
      <input type="radio" value="task--info" name="priority" /> Bassa
    </div>
    <div class="modal-action">
      <label for="addEv" class="btn btn-outline btn-error">Annulla</label>
      <label for="addEv" class="btn btn-outline btn-success" on:click={AddTodo}
        >Conferma</label
      >
    </div>
  </div>
</div>

<input type="checkbox" id="saveEv" class="modal-toggle" />
<div class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Sei sicuro di voler salvare?</h3>
    <br />
    <div class="modal-action">
      <label for="saveEv" class="btn btn-outline btn-error">Annulla</label>
      <label for="saveEv" class="btn btn-outline btn-success" on:click={save}>Conferma</label>
    </div>
  </div>
</div>

<input type="checkbox" id="RemEv" class="modal-toggle" />
<div class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Rimuovi un evento al calendario</h3>
    <br />
    <input type="text" bind:value={titleToDelete} class="input input-ghost w-full max-w-xs" placeholder="Titolo" />
    <br /><br />
    <input type="date" bind:value={dateToDelete} />
    <div class="modal-action">
      <label for="RemEv" class="btn btn-outline btn-error">Annulla</label>
      <label for="RemEv" class="btn btn-outline btn-success" on:click={Delete}>Conferma</label>
    </div>
  </div>
</div>
<style>
  .calendar-container {
    width: 90%;
    margin: auto;
    overflow: hidden;
    box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    background: #fff;
    max-width: 1200px;
  }
  .calendar-header {
    text-align: center;
    padding: 20px 0;
    background: #eef;
    border-bottom: 1px solid rgba(166, 168, 179, 0.12);
  }
  .calendar-header h1 {
    margin: 0;
    font-size: 18px;
  }
  .calendar-header button {
    background: #eef;
    border: 1px;
    padding: 6;
    color: rgba(81, 86, 93, 0.7);
    cursor: pointer;
    outline: 0;
  }
</style>
