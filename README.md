# finalexam_loop_engeering
**Core idea:** Pehle aap coding agent ko step-by-step prompts dete thay. **Loop Engineering** mein aap aisa system banate hain jo khud kaam **start, execute, check aur record** karta hai. Yani focus **prompt writing se loop design** par shift hota hai.
/////////////////////////////////////////////////In-session loop (/loop)
Fixed timer pe chalta hai — aap interval khud dete ho: /loop 5m check deployment status
Yeh sirf tab tak chalta hai jab tak session khula hai. Terminal band = loop band (yeh feature hai, bug nahi — ek casual loop session se zyada nahi chalna chahiye by default)
Ruknay ka faisla aap karte ho (cancel karke) — loop khud kabhi complete nahi hota, bas repeat karta rehta hai jab tak aap na rokein
Use case: "jab tak main dekh raha hun, kuch watch karo" — jaise deploy ya long test run
Middle option: --bg (background session) session band hone ke baad bhi chalta hai, lekin laptop on rehna zaroori hai
`/loop` kaam complete hone pe khud band **nahi** hota — sirf timer follow karta hai. Aapko manually cancel karna parta hai, chahe kaam ho chuka ho ya na ho. Sirf `/goal` khud rukta hai, jab condition sach ho jaye.
/loop 2m check if the website is live and tell me
Har 2 minute check hota rahega — "abhi nahi", "abhi nahi", phir "haan live ho gayi". Loop yahan bhi khud nahi rukega, aapko khud cancel karna hoga:
//////////////////////////////////////////////Conditional / run-until-done (/goal)
Timer nahi, condition pe based hai: "tests pass ho jayein" jaisi testable cheez
Ek alag checker (chota model) har turn ke baad transcript padh kar decide karta hai "kaam ho gaya ya nahi" — jis agent ne kaam kiya, woh khud apna check nahi karta
Condition poori hote hi loop khud ruk jata hai (koi manual cancel nahi chahiye)
Yeh bhi in-session hi hai — terminal band = yeh bhi ruk jayega, chahe kaam complete ho ya na ho
Koi built-in max-tries limit nahi hoti — agar chahiye to condition mein khud likhni padti hai (jaise "...ya 20 tries ke baad stop")
Use case: "jab tak kaam prove na ho jaye tab tak try karte raho" — jaise code fix karna jab tak tests pass na hon
/goal All tests in test/auth pass and npm run lint is clean.
Yahan agent khud code edit karega, tests chalayega, failures dekhega, dobara try karega — aur jaise hi checker confirm kare ke tests waqai pass ho gaye hain, loop khud ruk jayega. Aapko cancel nahi karna parta.
//////////////////////////////////////////Scheduled (Routine)
6. Scheduled (Routine) — clock pe chalta hai, laptop band ho tab bhi. Claude Code mein Routine cloud pe (Anthropic ke servers pe) chalta hai — aap prompt, repos, connectors, aur trigger (schedule/API call/GitHub event) set karte ho, phir yeh khud chalta rehta hai. Daily run limit hoti hai (Pro=5, Max=15, Team=25), aur default claude/ branches pe hi push karta hai — main pe nahi.
Haan, bilkul sahi samjha aapne — agar claude/fix-tests branch ka kaam theek lage, to aap use apne main branch (apne project folder ka asal code) mein merge kar lete hain. Yani
**Short example:** Har Monday subah 9 baje ek Routine chalta hai jo dependencies check karta hai aur safe updates ki `claude/update-deps` branch pe push kar deta hai. Aap dekh kar theek lage to `main` mein merge kar dete hain — laptop band ho ya on, Routine phir bhi chalta rehta hai kyunke woh Anthropic ke server pe hai, aapke computer pe nahi.
/////////////////////////////////////////. Event-driven
Event-driven — jaise doorbell, jab tak kuch hota nahi kuch chalta nahi. PR khule, message aaye, alert fire ho — tabhi reaction. Teen routes: GitHub events → Routine, chat message → Channel (isko live session chahiye, machine on honi zaroori), kuch bhi aur (API request) → Routine with API trigger.
**Real-world example:** Aap ek open-source project maintain karte hain. Jab bhi koi contributor pull request khole, GitHub event trigger hota hai aur Routine khud PR ko review kar ke comment kar deta hai — "yeh function test nahi ho raha" ya "code style theek hai". Koi bhi is PR ko kabhi na kholay, to Routine kabhi chalega hi nahi — bilkul doorbell ki tarah, sirf tab bajta hai jab koi button dabaye.
/////////////////////////////////////////////////////////Loop ki body
**********************************Isolation (worktrees)
ek loop ek se zyada agents ek sath chalata hai. Agar dono agent ek hi project ki files pe kaam kar rahe hon, to ek ka kaam dusre ka kaam overwrite kar sakta hai.
Iska hal: har agent ko apni alag copy-book de do — apna alag folder, jahan sirf wo likh raha hai. Isko kehte hain worktree. Yeh ek separate working folder hai, apni branch pe, lekin same project ki history share karta hai. Ek agent ka kaam dusre ke folder ko chhoo bhi nahi sakta.
sal masla yeh hai ke ek agent ka kaam dusre ka kaam mita ya bigaad sakta hai. Jaise agar dono ek hi file edit kar rahe hon, ek agent ki changes dusre ki changes ke upar likhi ja sakti hain, aur dono ka kaam kharab ho sakta hai.
**********************************Knowledge (skills)
har baar loop chalta hai to wo fresh session hota hai — usay tumhare project ki aadatein yaad nahi hoti. Bina help ke, wo har baar guess karta hai ya dobara pooch-taach karta hai, jo time aur tokens dono waste karta hai. Iska hal: ek skill — ek file (SKILL.md) jisme yeh sab likh diya jata hai, aur agent har run pe usay padh leta hai.
**********************************Action (connectors)
Agar loop bhi sirf files padh sakta hai, to wo sirf bata sakta hai — "yeh fix kar do." Wo khud PR khol nahi sakta, Slack pe message nahi bhej sakta, database update nahi kar sakta.
Connectors (MCP pe bane hue) yeh cheez badalte hain. Yeh loop ko karne dete hain — PR kholna, ticket update karna, Slack pe post karna. Farq yeh hai: ek loop jo sirf kehta hai "yeh fix hai," aur ek loop jo khud PR khol deta hai, ticket link kar deta hai, aur channel pe post kar deta hai — jab CI green ho jaye.
Ek chhoti si extra baat is concept ki: jab loop khud actions leta hai, to 3 cheezein zaroori ho jaati hain — (1) kam aur focused tools do, warna model confuse ho jata hai; (2) wo action safe-to-repeat ho (retry pe duplicate na bane, jaise dobara "create customer" se do customer na ban jayein); (3) error message clear ho, taake agla try khud theek ho sake.
*********************************Maker-checker (subagents)
Bilkul yehi masla hoti hai jab ek agent apna hi kaam check kare — wo apne aap ko bohot aasani se "pass" de deta hai. Isliye loop mein rule hai: jo agent kaam karta hai, wo hi usay approve nahi karega. Ek dusra agent (kabhi alag model bhi) us kaam ko check karta hai.
Isko maker-checker ya LLM-as-judge kehte hain.
**********************************Dynamic workflows**
ke sab kaam (worktree banana, skill padhna, connector se action, maker-checker se check) ek dafa ek script mein likh dena, taake agli baar sirf "yeh workflow chalao" bolna kaafi ho.
Zaroori baat: yeh workflow **ek dafa chal kar khatam ho jata hai aur bhool jata hai** — yeh sirf loop ka **body** hai, poora loop nahi.
Poora loop = **Heartbeat** (shuru karta hai) + **Workflow** (body, yehi hai) + **Progress file** (spine, jo yaad rakhta hai).
////////////////////////////////////////////////////////Verification skills
Ak agent say checking ka kam kis trhn krwana hy y Is check ko bhi ek skill mein likh diya ja sakta hai (jaisa Concept 9 mein seekha), taake agent khud yeh check kar sake, tumhe har baar khud dekhne ki zaroorat na pade.
******************************************************Four parts
Standalone — Yeh skill kisi aur cheez se judi nahi. Tum khud, jab jee chahe, bologe "yeh check chalao." Jaise tumhare paas ek separate button ho jo sirf tum dabao.
Embedded — Yeh skill kisi dusri skill ke andar fit ho jati hai. Jaise: koi skill code likhti hai, aur uske khatam hote hi, yeh check khud-ba-khud chal jata hai — bina tumhe alag se bolna pade. "Code likha → turant check bhi ho gaya."
Chained — Ek skill dusri skill ko khud call karti hai. Jaise: pehli skill kehti hai "mera kaam khatam, ab tum (dusri skill) chalo." Ek chain ban jati hai — A khatam → B shuru → B khatam → C shuru.
Every PR — Yeh sabse bada "ghar" hai: poori team ke liye. Jab bhi koi (chahe koi bhi ho) code submit kare (PR banaye), yeh check automatically chal jaye — har waqt, sab ke liye, koi bhi bhoole nahi.
/////////////////////////////////////////////////////////Spine
Yaad hai spine (Concept 12) ki baat ki thi — ke loop ko yaad rakhne ke liye disk pe kuch save karna padta hai? Asal mein spine do hisso mein bant'ta hai, do alag files:
1. Rules file (CLAUDE.md ya AGENTS.md)
Yeh wo file hai jisme hamesha ke liye kaam aane wali habits aur seekhein likhi jaati hain. Jaise: "hum yeh coding style follow karte hain," "yeh galti dobara mat karna." Yeh file har single run ke shuru mein padhi jaati hai — chahe kaam kuch bhi ho.
2. Progress file (progress.md)
Yeh wo file hai jisme roz ka status likha jata hai: "aaj yeh kaam kiya, yeh abhi baaki hai." Yeh roz badalti rehti hai.
Us intern wali example se yaad karo: rules file diary ke front mein likhi hui hamesha wali seekhein hain, aur progress file diary ke back mein likha roz ka update hai.
//////////////////////////////////////////////////////Concept — Teen feedback loops, teen speed ,Keeping Human Control
Socho tum ek bacchay ke liye ek typing game bana rahi ho:
Coding loop (minutes mein) — agent khud code likhta hai, test karta hai, bug fix karta hai — yeh wahi loop hai jo humne ab tak seekha.
Feedback loop (hours mein) — tum khud game khol kar dekhti ho, decide karti ho "buttons bade karo," phir agent ko naya instruction deti ho.
Outside loop (din mein) — asli log (jaise wo bacha) game use karte hain, aur unka react karna tumhe batata hai agla kya badalna hai.
Yeh teeno ek dusre ke andar rehte hain — chhota loop (coding) bade loop (feedback) ke andar chalta hai, aur wo dono sabse bade (outside) ke andar.
///////////////////////////////////////////////////Token cost hi asal limit hai
Loop ke sath bilkul yehi hota hai: loop kitni martaba chalta hai, yehi uski cost decide karta hai — command ka naam ya kaam ka size nahi. Ek example: agar loop din mein 5 dafa chale, to mahine ka kharcha kaafi kam (~$20) ho sakta hai. Wahi loop agar har 5 minute mein chale, to kharcha 100 guna zyada ($1000+) ho sakta hai — jabke kaam wahi hai.
Isay control karne ke tareeqe:
Har loop pe limit lagao (max tries, max time, max paisa)
Chhote kaam ke liye sasta model, mushkil check ke liye behtar model use karo
Loop ka prompt aur rules file chhota rakho — yeh har run pe cost hota hai
Loop ko kam baar chalao (jaisa zaroorat ho, na ke bar bar)
//////////////////////////////////////////////////Kaam check karna abhi bhi tumhara zimma hai
"Trust the loop to do the work" — loop pe bharosa karo ke wo kaam kar sake, code likh sake, test chala sake. Usay kaam karne do.
"Check the work before it counts" — lekin jab tak tum khud dekh kar confirm nahi karti ke kaam waqai sahi hai, tab tak usay "final" ya "done" mat samjho. Matlab: kaam sirf tab "count" hota hai — yani asal mein maana jata hai ke ho gaya — jab tumne khud check kar liya, sirf loop ke "PASS" bolne se nahi.
Simple example: socho loop ne code likha, checker ne bola "PASS," aur PR (pull request) ban gaya. Lekin agar tum yeh PR bina padhe seedha merge kar do (main project mein daal do), to yeh "count" ho gaya bina tumhare check kiye — jo risky hai. Sahi tareeqa: PR ko khud padho, dekho theek hai, phir merge karo — tabhi wo kaam "count" hota hai, asal mein complete mana jata hai.
To short mein: loop kaam kare, lekin final mohar tumhari honi chahiye, checker ki nahi.
****************************************************Concept — In the loop, On the loop, Out of the loop
**In the loop, On the loop, Out of the loop — concise:**
- **In the loop** → Har action se pehle insaan ki approval zaroori. (Slow, zyada control)
- **On the loop** → System khud kaam karta hai, insaan sirf dekhta hai, rok sakta hai. (Fast, zyada autonomy)
- **Out of the loop** → Koi nahi dekh raha, koi rok nahi sakta. **Hamesha galat.**
**Yaad rakhne wali baat:** Accha loop dono ka **mix** hota hai — safe kaam on-the-loop, risky kaam in-the-loop. Aur "on the loop" agar check karna band kar do, to chupke se **out-of-the-loop** ban jata hai.
//////////////////////////////////////////////////Apna project samajhna band mat karo
Loop jitna tez kaam karta hai, utna gap badhta hai: loop kya ship kar raha hai vs tum kitna samajhti ho. Isay AI gravity kehte hain — sab kuch dheere-dheere AI pe chhod dene ka pull.
Iska ilaaj simple hai: har hafte thoda waqt nikaal kar padho ke loop ne kya-kya badla/ship kiya. Isse:
Tumhe pata rehta hai project mein asal mein kya ho raha hai
Tum galti jaldi pakad loti ho, badi banne se pehle
Tum "engineer" rehti ho, sirf "button dabane wali" nahi........






















