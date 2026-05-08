// server.js
const express = require("express");
const path = require("path");
const bodyParser = require("body-parser");
const app = express();
const PORT = process.env.PORT || 3000;

// In-memory opslag van deelnemers
const participants = [];

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, "public")));

// API: lijst deelnemers
app.get("/api/participants", (req, res) => {
  res.json(participants);
});

// API: deelnemer toevoegen
app.post("/api/participants", (req, res) => {
  const { name, guess, objective, hobbies, mustdos, food } = req.body || {};
  if (!name || !guess || !objective || !hobbies || !mustdos || !food) {
    return res.status(400).json({ error: "Alle velden zijn verplicht." });
  }
  const participant = { name, guess, objective, hobbies, mustdos, food };
  participants.push(participant);
  res.status(201).json(participant);
});

// API: voorstel activiteitenplan
app.get("/api/plan", (req, res) => {
  if (participants.length === 0) {
    return res.json({ days: [], activitiesByDay: {} });
  }

  const weekendDays = [
    { date: "2026-05-22", label: "Vrijdag 22-05-2026" },
    { date: "2026-05-23", label: "Zaterdag 23-05-2026" },
    { date: "2026-05-24", label: "Zondag 24-05-2026" },
    { date: "2026-05-25", label: "Maandag 25-05-2026" },
  ];

  const hobbyActivities = [
    { keyword: "voetbal", activity: "Bezoek een lokale voetbalpub om samen een wedstrijd te kijken." },
    { keyword: "bier", activity: "Doe een proeverij van lokale craft bieren in een gezellige bar." },
    { keyword: "wandelen", activity: "Plan een stadswandeling langs interessante wijken en uitzichtpunten." },
    { keyword: "museum", activity: "Bezoek een bekend museum met een korte rondleiding." },
    { keyword: "uitgaan", activity: "Reserveer een avond in een populaire uitgaansbuurt met bars en clubs." },
    { keyword: "spa", activity: "Plan een relaxmoment in een spa of wellnesscentrum." },
  ];

  const foodActivities = [
    { keyword: "steak", activity: "Boek een tafel in een goed aangeschreven steak-restaurant." },
    { keyword: "tapas", activity: "Plan een avond met shared tapas in een levendige wijk." },
    { keyword: "sushi", activity: "Zoek een all-you-can-eat sushiplek voor een gezamenlijke avondmaaltijd." },
    { keyword: "burger", activity: "Bezoek een hippe burgerbar met speciale burgers en craft bier." },
    { keyword: "pizza", activity: "Plan een informele pizzanight in een lokale pizzeria." },
  ];

  const objectiveActivities = [
    { keyword: "ontspannen", activity: "Blok een middag voor chillen in een park of aan het water." },
    { keyword: "feesten", activity: "Plan een late avond met barhoppen en eventueel een club." },
    { keyword: "cultuur", activity: "Voeg een culturele tour toe langs historische plekken." },
    { keyword: "sport", activity: "Plan een sportieve activiteit zoals karten, padel of een escape room." },
    { keyword: "eten", activity: "Maak een foodtour langs verschillende lokale eettentjes." },
  ];

  const genericMustDos = [
    "Plan een gezamenlijke groepsfoto op een herkenbare maar niet te herkenbare plek.",
    "Reserveer één verrassingselement waar niemand de details van weet.",
    "Laat ruimte in de planning voor spontane ideeën van de groep.",
  ];

  const allObjectives = participants.map(p => p.objective.toLowerCase()).join(" ");
  const allHobbies = participants.map(p => p.hobbies.toLowerCase()).join(" ");
  const allMustDos = participants.map(p => p.mustdos.toLowerCase()).join(" ");
  const allFood = participants.map(p => p.food.toLowerCase()).join(" ");

  const selectedActivities = [];

  function addIfMatch(list, text) {
    list.forEach(item => {
      if (text.includes(item.keyword) && !selectedActivities.includes(item.activity)) {
        selectedActivities.push(item.activity);
      }
    });
  }

  addIfMatch(hobbyActivities, allHobbies);
  addIfMatch(foodActivities, allFood);
  addIfMatch(objectiveActivities, allObjectives);

  genericMustDos.forEach(act => {
    if (!selectedActivities.includes(act)) selectedActivities.push(act);
  });

  if (selectedActivities.length < weekendDays.length * 2) {
    [
      "Plan een gezamenlijke brunch om de dag rustig te starten.",
      "Laat een blok vrij voor vrije tijd of kleine groepjes.",
      "Organiseer een korte quiz of spelavond over de groep."
    ].forEach(a => {
      if (!selectedActivities.includes(a)) selectedActivities.push(a);
    });
  }

  const activitiesByDay = {};
  weekendDays.forEach(d => (activitiesByDay[d.date] = []));
  let index = 0;
  selectedActivities.forEach(act => {
    const day = weekendDays[index % weekendDays.length].date;
    activitiesByDay[day].push(act);
    index++;
  });

  res.json({ days: weekendDays, activitiesByDay });
});

app.listen(PORT, () => {
  console.log(`Mannenweekend app draait op http://localhost:${PORT}`);
});

