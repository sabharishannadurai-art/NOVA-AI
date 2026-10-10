
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Nova AI</title>
<script src="https://js.puter.com/v2/"></script>
<style>
*{box-sizing:border-box}
body{
  margin:0;background:#10101c;color:#f5f3ff;
  font-family:system-ui,sans-serif
}
.app{max-width:760px;margin:auto;padding:20px;min-height:100vh;
display:flex;flex-direction:column}
header{display:flex;align-items:center;gap:12px;margin-bottom:28px}
.logo{background:linear-gradient(135deg,#a477ff,#588cff);
padding:12px;border-radius:14px;font-size:24px}
h1{font-size:23px;margin:0}
.sub{color:#aaa7c4;font-size:13px}
.welcome{margin:35px 0 25px}
.welcome h2{font-size:38px;line-height:1.15;margin-bottom:8px}
.gradient{background:linear-gradient(90deg,#b99aff,#79d4ff);
color:transparent;background-clip:text}
.tabs{display:grid;grid-template-columns:repeat(3,1fr);gap:8px}
button{border:0;border-radius:12px;padding:13px 8px;
background:#29273e;color:white;font-size:14px;cursor:pointer}
button.active{background:#6045a4}
#messages{display:flex;flex-direction:column;gap:12px;
padding:20px 0}
.msg{padding:13px 15px;border-radius:16px;
white-space:pre-wrap;overflow-wrap:anywhere;line-height:1.6}
.user{align-self:flex-end;background:#6045a4;max-width:90%}
.bot{align-self:flex-start;background:#211f32;max-width:100%}
.bot img{display:block;max-width:100%;height:auto;border-radius:12px;margin-top:10px}
.composer{margin-top:auto;position:sticky;bottom:0;
background:#10101c;padding:12px 0}
.input{display:flex;gap:8px;background:#211f32;
border:1px solid #3d3958;padding:8px;border-radius:16px}
textarea{flex:1;min-width:0;resize:vertical;min-height:45px;
max-height:130px;background:transparent;color:white;
border:0;padding:10px;font:inherit;outline:none}
#send{background:linear-gradient(135deg,#9c74ff,#567fff);
font-weight:bold}
#send:disabled{opacity:.5}
#status{font-size:12px;color:#aaa7c4;padding:8px 2px}
.hidden{display:none}
</style>
</head>
<body>
<div class="app">
<header>
  <div class="logo">✦</div>
  <div><h1>Nova AI</h1><div class="sub">Your AI. Your imagination.</div></div>
</header>

<section class="welcome" id="welcome">
  <h2>Hey there!<br><span class="gradient">What can I help with?</span></h2>
  <p class="sub">Ask questions, learn something new, or create an image.</p>
</section>

<nav class="tabs">
  <button class="active" data-mode="chat">💬 Chat</button>
  <button data-mode="study">📚 Homework</button>
  <button data-mode="image">🎨 Images</button>
</nav>

<main id="messages"></main>
<div id="status">Ready when you are.</div>

<div class="composer">
  <div class="input">
    <textarea id="prompt" placeholder="Message Nova AI..."></textarea>
    <button id="send">Send ↑</button>
  </div>
  <p class="sub">AI can make mistakes. Check important answers.</p>
</div>
</div>

<script>
const $=id=>document.getElementById(id);
let mode="chat",busy=false,history=[];

const instructions={
 chat:"You are Nova AI, a friendly, helpful assistant. Answer clearly and honestly. Admit when you are unsure.",
 study:"You are Nova AI, a patient homework tutor. Explain school subjects step by step with simple examples. Help students understand instead of just copying answers."
};

function bubble(text,kind="bot"){
 const el=document.createElement("div");
 el.className="msg "+kind;
 el.textContent=text;
 $("messages").appendChild(el);
 el.scrollIntoView({behavior:"smooth",block:"nearest"});
 return el;
}

function setMode(next){
 mode=next;
 document.querySelectorAll("[data-mode]").forEach(b=>
   b.classList.toggle("active",b.dataset.mode===mode));
 $("prompt").placeholder=mode==="image"
  ?"Describe an image to create..."
  :mode==="study"?"Ask your homework question...":"Message Nova AI...";
 $("status").textContent=mode==="image"
  ?"Describe the picture you want."
  :mode==="study"?"Homework tutor is ready.":"Chat is ready.";
}
document.querySelectorAll("[data-mode]").forEach(b=>
 b.addEventListener("click",()=>setMode(b.dataset.mode)));

$("send").addEventListener("click",send);
$("prompt").addEventListener("keydown",e=>{
 if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();send();}
});

async function send(){
 if(busy)return;
 const text=$("prompt").value.trim();
 if(!text)return;

 $("prompt").value="";
 $("welcome").classList.add("hidden");
 bubble(text,"user");
 busy=true;
 $("send").disabled=true;
 $("status").textContent="Nova is working...";

 try{
   if(mode==="image"){
     const result=await puter.ai.txt2img(text);
     const el=bubble("Your generated image:");
     if(result instanceof HTMLImageElement){
       el.appendChild(result);
       result.alt=text;
     }else if(result instanceof HTMLElement){
       el.appendChild(result);
     }else{
       throw new Error("Unexpected image response. Please try again.");
     }
     $("status").textContent="Image request complete.";
   }else{
     const messages=[
       {role:"system",content:instructions[mode]},
       ...history,
       {role:"user",content:text}
     ];
     const result=await puter.ai.chat(messages);
     let answer;
     if(typeof result==="string")answer=result;
     else{
       const m=result?.message??result;
       answer=typeof m?.content==="string"?m.content:
         Array.isArray(m?.content)?m.content.map(x=>x.text||"").join("\n"):
         m?.text||"";
     }
     if(!answer)throw new Error("No answer received.");
     bubble(answer);
     history.push({role:"user",content:text},
                  {role:"assistant",content:answer});
     if(history.length>16)history=history.slice(-16);
     $("status").textContent="Done.";
   }
 }catch(err){
   console.error(err);
   bubble("Sorry, that didn't work. Check your internet connection and AI service access. If asked, sign in to Puter. Error: "+(err.message||"Unknown error"));
   $("status").textContent="Request failed.";
 }finally{
   busy=false;
   $("send").disabled=false;
 }
}
</script>
</body>
</html>
