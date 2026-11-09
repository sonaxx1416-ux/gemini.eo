# gemini.eo

كلّم Gemini من لغة **eo**. مكتوبة بـ eo فوق `http()`، بدون أي تعديل في اللغة.

```eo
import gemini

var conver = gemini.Conversation("مفتاحك")
var s = input "Enter ask : "
var reply = conver.ask s
print h"Gemini : ${reply}"
```

## المفتاح

1. خذ مفتاح مجاني من https://aistudio.google.com/apikey (**Create API key**).
2. لا تكتبه في الكود. حطه في ملف `.env` جنب ملفك:
   ```
   GEMINI_API_KEY=المفتاح
   ```
3. واقرأه كذا:
   ```eo
   load_env()
   var conver = gemini.Conversation(env("GEMINI_API_KEY"))
   ```

## Conversation

المحادثة **تتذكر** الكلام اللي قبل، فتقدر تسأل "وش قصدك؟" ويفهم.

| | |
|---|---|
| `conver.ask "سؤال"` | يرجع الرد كنص. لو صار خطأ يرجع `""` والسبب في `conver.error` |
| `conver.system = "..."` | تعليمات: شخصيته، لغته، وش يجاوب ووش لا |
| `conver.temperature = 0.2` | من 0 لين 2: الصغير دقيق، والكبير مبدع (الافتراضي 0.7) |
| `conver.max_tokens = 300` | أقصى طول للرد |
| `conver.memory = 40` | كم رسالة يتذكر (الأقدم تنمسح) |
| `conver.clear` | يمسح المحادثة ويبدأ من جديد |
| `conver.count()` | كم رسالة في المحادثة |
| `conver.save "chat.json"` ، `conver.load "chat.json"` | تحفظ المحادثة وترجع لها بعد ما تقفل البرنامج |
| `conver.transcript()` | المحادثة كنص |
| `conver.model` | الموديل اللي رد آخر مرة |
| `conver.models = ["gemini-2.5-flash"]` | الموديلات بالترتيب |
| `conver.log = false` | يسكّت رسائل `[gemini]` |

## سؤال واحد

```eo
print gemini.ask(key, "وش عاصمة اليابان؟")
print gemini.ask(key, "translate: صباح الخير", "Reply in English only")
```

## الأخطاء

المكتبة **ما توقف برنامجك**. ترجع `""` وتطبع السبب:

| الرسالة | يعني |
|---|---|
| `no API key` | ما حطيت مفتاح |
| `wrong API key` | المفتاح غلط |
| `... is busy (429), retrying...` | خلص الحد المجاني للموديل، ينتظر ويجرب الموديل اللي بعده لحاله |
| `the question was blocked` | Gemini رفض السؤال |
| `can't reach Gemini` | ما فيه نت |

```eo
var reply = conver.ask s
if reply == "" {
    print "ما قدر يرد:", conver.error
}
```

## مع discord.eo

```eo
import discord
import gemini

load_env()
var bot = discord.Bot(env("DISCORD_TOKEN"))
var chats = {}

bot.command "ask", lambda msg, args {
    if not has(chats, msg.author.id) {
        var c = gemini.Conversation(env("GEMINI_API_KEY"))
        c.system = "أنت مساعد سيرفر ديسكورد، رد باختصار."
        chats[msg.author.id] = c
    }
    msg.channel.typing
    var reply = chats[msg.author.id].ask(join(args, " "))
    if reply == "" {
        reply = "ما قدرت أرد الحين 🙏"
    }
    msg.reply slice(reply, 0, 1900)
}

bot.run
```
