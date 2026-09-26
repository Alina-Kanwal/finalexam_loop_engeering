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



















