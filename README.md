# finalexam_loop_engeering
**Core idea:** Pehle aap coding agent ko step-by-step prompts dete thay. **Loop Engineering** mein aap aisa system banate hain jo khud kaam **start, execute, check aur record** karta hai. Yani focus **prompt writing se loop design** par shift hota hai.
/////////////////////////////////////////////////In-session loop (/loop)
Fixed timer pe chalta hai — aap interval khud dete ho: /loop 5m check deployment status
Yeh sirf tab tak chalta hai jab tak session khula hai. Terminal band = loop band (yeh feature hai, bug nahi — ek casual loop session se zyada nahi chalna chahiye by default)
Ruknay ka faisla aap karte ho (cancel karke) — loop khud kabhi complete nahi hota, bas repeat karta rehta hai jab tak aap na rokein
Use case: "jab tak main dekh raha hun, kuch watch karo" — jaise deploy ya long test run
Middle option: --bg (background session) session band hone ke baad bhi chalta hai, lekin laptop on rehna zaroori hai
