# Simulador 3D de Motor Homopolar

Simulador interativo em 3D do motor homopolar clássico: uma pilha AA em pé sobre um ímã de neodímio, com um laço de fio de cobre que fecha o circuito do terminal positivo até a borda do ímã.

## ▶ https://obb199.github.io/simulador-motor-homopolar/

Página única, sem build. Usa [three.js](https://threejs.org/) 0.178 via CDN com import map.

## Como funciona o motor

A corrente sai do terminal positivo, desce pelos **dois** ramos do laço em paralelo e chega à lateral do ímã, voltando por ele até a base da pilha. Sobre cada trecho vertical de fio age a força de Lorentz:

$$\vec{F} = I \vec{L} \times \vec{B}$$

Com $\vec{L}$ vertical, é a **componente radial** de $\vec{B}$ que produz força tangencial — não o campo total. E ela aponta no mesmo sentido de giro nos dois lados, porque tanto $\vec{L}$ quanto $\hat{r}$ trocam de sinal de um ramo para o outro. Por isso os dois ramos somam torque em vez de se cancelarem.

Com o polo norte para cima e o "+" para cima, o giro é **horário visto de cima**. Inverter o ímã troca o sentido sem mexer na corrente; inverter a pilha troca os dois; inverter ambos devolve o giro original.

## Modelo físico

O simulador integra o sistema a cada quadro, em subpassos fixos de 2 ms.

### Circuito

A corrente cai com a força contraeletromotriz e com a resistência de contato, onde $q$ é a qualidade do contato:

$$I = \frac{\varepsilon - \text{FCEM}}{R_\text{interna} + R_\text{fio} + R_\text{contato}}$$

$$R_\text{contato} = 0.02 + 0.14 \cdot \frac{1 - q}{q}$$

### Torque e FCEM

Mesma constante de motor $k_t$ nos dois, como manda o SI:

$$\tau = k_t I \qquad \text{FCEM} = k_t \omega \qquad k_t = 2 (f_r B) \ell r$$

### Rotação

Dinâmica real, com inércia e três regimes de atrito — seco, viscoso e arrasto aerodinâmico:

$$J \frac{d\omega}{dt} = \tau - \text{sgn}(\omega) \left( \tau_c + b |\omega| + c \omega^2 \right)$$

Se $\tau \le \tau_c$ o rotor fica **travado**, mesmo com corrente passando — o que acontece de verdade com ímã fraco ou contato ruim.

### Temperatura

O calor da resistência interna fica na pilha; o fio só sente o que é dissipado nele e nos contatos, e girar aumenta a convecção:

$$C \frac{dT}{dt} = I^2 (R_\text{fio} + R_\text{contato}) - (k_0 + k_1 |\omega|)(T - T_\text{amb})$$

### Constantes

As constantes não têm todas o mesmo peso de evidência, então cada uma está marcada com a sua procedência. A mesma classificação está no código-fonte, em `index.html`, no objeto `PHYS`.

| Procedência | Significado |
|---|---|
| **MEDIDO** | valor típico de literatura ou datasheet do componente |
| **CALCULADO** | deduzido da geometria do modelo ou de propriedade do material |
| **ESTIMADO** | ordem de grandeza defensável, sem medição direta |
| **CALIBRADO** | ajustado para reproduzir o comportamento observado em motores reais |

| Parâmetro | Valor | Procedência | Origem |
|---|---|---|---|
| $R_\text{interna}$ | 0,25 Ω | MEDIDO | uma AA alcalina nova fica entre 0,15 e 0,3 Ω |
| $I_\text{max}$ | 6 A | MEDIDO | curto-circuito de uma AA não passa disso |
| $R_\text{contato}$ (piso) | 0,02 Ω | ESTIMADO | contato cobre-aço, mesmo bem apertado |
| $R_\text{contato}$ (escala) | 0,14 Ω | **CALIBRADO** | leva o contato de ~0,02 Ω a ~0,58 Ω |
| $f_r$ (fração radial de $B$) | 0,28 | ESTIMADO | componente radial junto à borda do ímã de disco |
| $\ell$ | 12 mm | ESTIMADO | o campo cai com ~$1/r^3$, só o trecho vizinho conta |
| $r$ | 14 mm | CALCULADO | raio do laço na escala do modelo (ímã de 24 mm) |
| ramos | 2 | EXATO | topologia do circuito |
| $J$ | 4,0 × 10⁻⁷ kg·m² | CALCULADO | $J \approx m r^2$, com ~2 g de cobre a 14 mm |
| $\tau_c$ | 2,0 × 10⁻⁵ N·m | **CALIBRADO** | atrito seco da ponta sobre o terminal |
| $b$ | 8,0 × 10⁻⁷ N·m·s | **CALIBRADO** | atrito viscoso, domina em rotação baixa |
| $c$ | 6,5 × 10⁻⁹ N·m·s² | **CALIBRADO** | arrasto, domina acima de ~1000 rpm |
| $C$ | 0,77 J/K | CALCULADO | 2 g de cobre × 385 J/(kg·K) |
| $k_0$ | 0,020 W/K | **CALIBRADO** | convecção natural com o fio parado |
| $k_1$ | 8,0 × 10⁻⁵ W/K por rad/s | **CALIBRADO** | convecção forçada pela rotação |
| $T_\text{amb}$ | 25 °C | CONVENÇÃO | temperatura ambiente de sala |

### Sobre os seis valores calibrados

Os coeficientes de contato, de atrito e de resfriamento não têm como ser derivados de primeiros princípios: dependem de como o arame foi dobrado à mão, do acabamento do terminal onde a ponta se apoia, de quanto o laço aperta o ímã e do formato exato que o fio ficou. Foram ajustados **em conjunto** para que o simulador reproduza quatro fatos observáveis do experimento real:

1. nos valores padrão o motor gira a ~1.290 rpm, dentro da faixa de centenas a poucos milhares de rpm que esses motores atingem;
2. no ajuste mais favorável chega a ~3.350 rpm, e não além disso;
3. no ajuste mais desfavorável não parte — ímã fraco somado a contato ruim e atrito alto realmente não vence o atrito estático;
4. o fio passa de 70 °C em ~30 s, condizente com esses fios ficarem quentes demais para tocar em menos de um minuto.

Mexer nesses seis números muda a rotação e o aquecimento que o simulador mostra. **Não muda nenhuma das relações físicas entre as grandezas** — quem define o comportamento qualitativo são as equações, não a calibração.

### Ordens de grandeza que o modelo produz

Nos valores padrão (1,5 V · 0,80 T · contato 80% · 0,15 Ω · atrito 1,0×):

| Grandeza | Valor |
|---|---|
| Corrente | 3,27 A |
| Tensão nos terminais | 0,68 V (de 1,5 V em aberto) |
| FCEM | 10 mV |
| Torque | 0,25 mN·m |
| Rotação | ~1.290 rpm |
| Potência dissipada | 4,9 W |
| Temperatura do fio | 49 °C em 10 s · 75 °C em 30 s |

A queda de 1,5 V para 0,68 V nos terminais não é erro: é o que uma alcalina realmente faz em quase-curto. E a FCEM de 10 mV contra 1,5 V mostra por que ela é desprezível neste motor — quem limita a rotação é o atrito, não a indução.

## Segurança no experimento real

O circuito é praticamente um curto. A corrente passa de 3 A e o fio chega a mais de 100 °C em menos de um minuto: mantenha o contato por poucos segundos e segure o fio só pela parte fria. Evite baterias de lítio, que entregam corrente muito maior e podem entrar em fuga térmica. Mantenha ímãs de neodímio longe de eletrônicos, cartões magnéticos e crianças pequenas.

## Rodando localmente

O import map exige HTTP — abrir o arquivo direto com `file://` não funciona.

```bash
python3 -m http.server 8000
# abra http://localhost:8000
```
