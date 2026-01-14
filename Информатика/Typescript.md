
**TypeScript** — это язык программирования, который является **надмножеством JavaScript**. Это означает, что любой корректный JS-код является корректным TypeScript-кодом, но TypeScript добавляет дополнительные возможности, главная из которых — **статическая типизация**.
TypeScript — это **строго типизированный язык программирования с открытым исходным кодом**, разработанный Microsoft в 2012 году. Он является **надмножеством (superset) JavaScript**, что означает:

- Любой корректный JavaScript-код является корректным TypeScript-кодом
    
- TypeScript добавляет статическую типизацию и дополнительные возможности
    
- Код компилируется (точнее, транспилируется) в JavaScript
    

**Ключевой принцип:** "TypeScript — это JavaScript, который масштабируется"

```Typescript
let words: string[] = ["Да.", "Нет.", "Возможно"];

let words_len: number = words.length;

let answer_index: number = Math.floor(Math.random() * words_len);

  

prompt("Задайте вопрос!");

console.log(words, words_len, answer_index);

alert(words[answer_index]);
```

