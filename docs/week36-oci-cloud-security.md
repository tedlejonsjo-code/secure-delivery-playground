# Week 36 OCI Cloud Security Lab
## 1. Min OCI-miljö/Min lokala Linux-miljö/Min lokala WSL-miljö
- Tenancy (N/A): tedlejonsjo
- Compartment (Endast OCI):tedlejonsjo (root)
- Region (Endast OCI): eu-stockholm-1
- Availability Domain (Endast OCI): AD-1
- VM-namn/hostnamn/WSL-maskinnamn: tedklar-vm
- Operativsystem: Canonical Ubuntu
- Shape (Hårdvara, gäller alla): VM.Standard.E2.1.Micro
- Inloggningsmetod: SSH via WSL
---
## 2. Linux-kommandon
| Kommando | Vad visar det? | CIA-koppling |
|-----------|-----------|-----------|
| whoami | Visar användarnman |CIA Koppling till Konfidentalitet|

 hostname | VM namnet = tedklar-vm i detta fall |CIA Koppling till Tillgänglighet |

| pwd |Vart du är i systemet vilket i detta fall är /Home/ubuntu |CIA Koppling till tillgänglighet |

| uname -a |Visar operativsystemet du är på |CIA Koppling till tillgänglighet|

| uptime |Hur länge servern har varit igång = 1 dag, 22 tim, 33 min vid dokumentation|CIA Koppling till tillgänglighet|

|id|Visar användar grupp och numeriska id. I detta fall är vårat user ID (ubuntu)=1001, Grupp nummer 1001 och våra icke primära gruppnummer 1001.|CIA Koppling till konfidentalitet|

|groups|Visar grupperna för varje användare|CIA Koppling till konfidentalitet|

|ps aux | head| Visar alla processer med deras ägande användare, proccessor ID, cpu använding i %, minnesanvänding i %, residens storlek.|CIA Koppling till tillgänglighet|

---
## 3. Hardening
| Kontroll | Risk | Vad gjorde jag? | Hur verifierade jag? | CIA |
|-----------|-----------|-----------|-----------|-----------|
| Kontroll 1 - Identitet och behörigheter |En risk om man inte kontrollerar med denna kontroll är att en person utan behörigheter har kommit in i systemet via ditt konto och kan ändra på rättigheter och så vidare, till exempel personal inom branchen eller angripare, om man inte kontrollerar sin egna åtkomskt kan sitt konto missbrukas ännu mer om någon får tag på det.| Jag körde Whoami, id och groups.|Jag verifierade genom att jag körde Whoami, id och groups och såg att jag hade rätt åtkomst och var med i rätt grupper| I CIA tråden så är detta konfidentalitet|

|Kontroll 2 - Filrättigheter|Risken är att om man inte kör kontollen är att någon har rättigheter till filer som dom inte borde ha till exempel personal och angripare.|Kollade filrättigheter via ls -l test.txt, ändrade rättigheter via chmod 600 test.txt, kontrollerade igen via ls -l test.txt|Jag verifierade resultatet genom att köra ls -l igen och såg att test.txt rättigheter hade ändrats.|Detta kopplas till cia tråden integritet| 

|Kontroll 3 - Systemuppdateringar|Risken om man inte kollar är att angripare kan komma in i systemet och därför behövs mer uppdateringar av systemet|Kollade om det fanns systemuppdateringar via sudo apt update och apt list --upgradable|jag körde sudo apt update och sedan apt list --upgradable för att see uppdateringarna och då kom dom upp i en lista. |Denna del kopplas till Konfidalitetsdelen av CIA tråden.|

|Kontroll 4 - Processer|Risken som sker om man inte kollar proccesser är att man missar en oväntad proccess som kan leda till att servern inte funkar som den ska eller är långsammare vilket gör den mer otillgänglig.|Kollade vilka proccessor som va igång via ps aux | head kommandot|Jag veriferade genom att jag skrev ps aux | head och såg alla proccesser och om något var oväntat.|I CIA tråden kopplas det till tillgänglighet|

|Kontroll 5 - Loggar |Om man inte kollar loggar riskerar man att missa ändringar i systemet och filer av obehöriga.|Kollade loggar i VM via journalctl -n 20|Jag verifierade genom att köra kommandot och sedan fick jag upp dom senaste 20 loggraderna och såg att inget misstänkt hade sket i loggarna| I CIA tråden kopplas denna kontroll till integritet|

|Kontroll 6 - SSH|Om SSH konfigurationen är dålig så kan obehöriga få åtkomst till servern|Jag beskrev hur jag loggade in, varför SSH används och vad jag skulle göra om SSH slutade fungera|Jag verifierade genom att jag prövade med att logga in via ssh och det funkade och körde whoami och hostname för att verifiera att jag var inne på VM med rätt användare.|Denna del kopplas till Konfidentalitet i CIA tråden|

---
## 4. Recovery-plan
### Vad kan gå fel? Ssh funkar inte om nycklar, konto eller brandväg påverkas. Om man har för stränga brandväggsregler så kan tillgänglighet påverkas. Start eller tjänst kan också betee sig oväntat om det blir till exempel fel vid uppstart, tjänster och scripts.

### Hur upptäcker jag problemet? Jag upptäcker problemet via error meddelanden, loggar, status på VM, serial output till exempel.

### Vad kontrollerar jag först? kontrollera och see om VM körs, vilken zon den är och vilket project den är i.

### Hur återställer jag åtkomst? Via användning av snapshots, det gör man via boot volume backups i oracle cloud.

### När behöver jag hjälp? Om du inte kan återskapa VM.
---
## 5. Backup
### Vad har jag sparat? Jag har sprarat snapshot, rapport, reflektion, evidens och scripts.
### Vad finns i GitHub? Rapport, evidens, reflektion och scripts.
### Vad kan återskapas? Du kan återskapa en version av din vm instans via att göra en ny instans baserat på din snapshot. Du kan återskapa deras inställningar och filer via snapshots och Rapport, evidens, reflektion och scripts via github.
### Vad går inte att återskapa? SSH nyckel, din instansen mellan att VM:et försvann och din snapshot. Din OCID och specifica metadata, Sjävla instanen går inte att återskapa utan du måste skapa en ny baserat på din boot volume.
---
## 6. Cleanup
### VM-instans - jag stoppade min VM instans och kollade så att den är på rätt status.  
### Diskar - Jag kontrollerade om det fanns några andra boot-diskar än den jag använde och det fannns det inte
### Backuper - Jag kontrollerade snapshots och behållde endast den jag hade. Man ska göra detta för att spara kostnads och lagringsresurser, det är lättare att hitta och det är en säkerhetsrisk eftersom dom kan innehålla information som inte längre ska sparas.
### Publika IP-adresser - Jag kollade om jag hade en statisk eller tillfällig public ip adress och såg att jag hade en tillfällig och jag kontrollerade reserverade ipv4 adresser på ip manager och hittade inget.
### GitHub-evidens - Jag kontrollerade att Github rapporten och evidensen fanns i Github och att inga lösernord eller känslig data.
---
## 7. CIA-reflektion
### Konfidentialitet = Rätt person får åtkomskt till din vm, I denna labb har det framkommit via kontroll 2 där jag Kollade filrättigheter via ls -l och därmed kan kontrollera att ingen har filrättigheter till vad dom inte borde ha. Gjorde också kontroll 1 som visade vem jag är så att man har koll på det via whoami kommandot, sen körde jag id som visar vilket användar och grupp id:n användaren tillhör i systemet, kontroll 3 vilket var att kolla om det fanns uppdateringar vilket är bra för att förhindra att angripare får åtkomskt.

### Integritet = Handlar om att information och system inte ändras felaktigt eller obemärkta. I denna labb har det synts via kontroll 5 som visar loggar av dom senaste 20 loggraderna så jag kan see viss misstänkt aktivet i systemet.

### Tillgänglighet = Handlar om att systemet går att använda det när det behövs. I labben har tillgänglighet synts via till exempel kontroll 4 där jag skrev ps aux | head som kontrollerar processer vilket gör att jag kan see om något verkar påverka systemet, uname -a vilket visar vilket operativsystem du är i och som används för att see att du är i rätt operativsystem som VM:et ska vara tillgänglig för, pwd som visar vart du är i systemet så att du kan see att VM:et är på rätt ställe och hostname som visar vilket VM du använder och att du är i rätt VM.

## 8. Reflektion
### Vad fungerade bra? Vad som fungerade bra var att göra kontrollerna, linux kommandorna och CIA reflektionen.
### Vad var svårt? Den svåraste delen kom genom att göra Backup och Recovery plan.
### Vad lärde jag mig? Jag har lärt mig en hel del. Men främst har jag lärt mig hur man skapar en vm, Gör en snapshot så att den kan återställas vid fel och har en recovery plan då också. Jag har lärt mig vad vissa kommandon som används för cybersäkerhet gör och jag har lärt mig kontroller för att see till att ingen har åtkomst till Vm:et, att dom har rätt åtkomskt om dom får VM tillgång. Att filer inte har ändrats utav någon obehörig och att systemet är tillgänglig när den behövs. Jag har också lärt mig hur man städar upp efter sitt VM via cleanup.