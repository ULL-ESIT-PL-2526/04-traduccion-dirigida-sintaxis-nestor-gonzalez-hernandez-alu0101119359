# Informe de Práctica 4 
# Traducción dirigida por sintaxis

## 1. Setup

Realizados los comandos pertinentes, generado parser, pasan todos los test.

## 2. Lexer

1. Describa la diferencia entre /* skip whitespace */ y devolver un token.

/* skip whitespace */ realmente no toma ninguna acción, con lo que procesa el caracter sin realizar ningún cambio al resultado final. No devulve un token.

2. Escriba la secuencia exacta de tokens producidos para la entrada 123**45+@.

NUMBER, OP, NUMBER, OP, INVALID, EOF

3. Indique por qué ** debe aparecer antes que [-+*/].

La segunda regla captura el string *, que es un subconjunto de **. 
Las reglas se procesan en orden de aparición.
Si [-+*/] apareciera antes que **, esta siempre capturaría el subconjunto, siendo imposible que ** capture.

4. Explique cuándo se devuelve EOF.

Se devuelve EOF al llegar al final de la entrada.

5. Explique por qué existe la regla . que devuelve INVALID.

Sin esta regla se crearía un error no definido en el proceso lexico al apararecer un caracter no entendido por el lenguaje.
Con esta regla, el caracter es captado y crea error en el proceso de parse. 
El error lo crea un token INVALID, con lo cual queda muy claro que se trata de un fallo de input.

## 3. Saltar comentarios

Añadido tras el skip de espacios blancos
```jison
"//".+                { /* skip commented */;  }
```

## 4. Punto flotante

Modificado 
```jison
[0-9]             { return 'NUMBER';       }
```
a
```jison
[0-9]+((\.[0-9]+)|\.)?([eE][-+][0-9])?             { return 'NUMBER';       }
```

## 5. Pruebas

Añadido a grupo de tests ```describe('Basic number parsing', () => {```
```js
test('should handle floating point', () => {
  expect(parse(" 1.5")).toBe(1.5);
  expect(parse("2.5e-2")).toBe(0.025);
  expect(parse("2.5E+2")).toBe(250);
  expect(parse("2.")).toBe(2);
});
```
 y
 ```js
describe('Comments', () => {
  test('should handle comments', () => {
    expect(parse("// 1.5 + 2 \n3+3")).toBe(6);
    expect(parse("//x +y\n 1+3\n")).toBe(4);
    expect(parse("//x +y\n 1+3\n//12+x")).toBe(4);
  });
});
```
como su propio grupo.

a los respectivos grupos (según operandos) añadidos los test
```js
expect(parse("0.2 + 0.6")).toBe(0.8);
expect(parse("2e-2 + 3.5e+2")).toBe(350.02);
expect(parse("0.2 - 0.6")).toBeCloseTo(-0.4);
expect(parse(" 1e+2 - 2e-2 ")).toBeCloseTo(99.98);
expect(parse("0.5 * 0.5")).toBeCloseTo(0.25);
expect(parse("1/ 0.1")).toBeCloseTo(10);
expect(parse("2 ** 0.5")).toBeCloseTo(1.41421);
```

En grupo ```'Input validation and error cases'``` eliminado
```js
expect(() => parse("3.5")).toThrow();
```
y añadido
```js
expect(() => parse("3..2")).toThrow();
expect(() => parse("3.1.2")).toThrow();
expect(() => parse("3..")).toThrow();
expect(() => parse("3.2.e+2")).toThrow();
```