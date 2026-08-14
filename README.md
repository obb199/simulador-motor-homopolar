# Simulador 3D de Motor Homopolar

Simulador interativo em 3D do motor homopolar clássico: uma pilha AA em pé sobre um ímã de neodímio, com um laço de fio de cobre que fecha o circuito do terminal positivo até a borda do ímã.

**[▶ Abrir o simulador](https://obb199.github.io/simulador-motor-homopolar/)**

Página única, sem build. Usa [three.js](https://threejs.org/) 0.178 via CDN com import map.

## Como funciona o motor

A corrente sai do terminal positivo, desce pelos **dois** ramos do laço em paralelo e chega à lateral do ímã, voltando por ele até a base da pilha. Sobre cada trecho vertical de fio age a força de Lorentz:

$$\vec{F} = I\,\vec{L} \times \vec{B}$$

Com $\vec{L}$ vertical, é a **componente radial** de $\vec{B}$ que produz força tangencial — não o campo total. E ela aponta no mesmo sentido de giro nos dois lados, porque tanto $\vec{L}$ quanto $\hat{r}$ trocam de sinal de um ramo para o outro. Por isso os dois ramos somam torque em vez de se cancelarem.

Com o polo norte para cima e o "+" para cima, o giro é **horário visto de cima**. Inverter o ímã troca o sentido sem mexer na corrente; inverter a pilha troca os dois; inverter ambos devolve o giro original.

## Modelo físico

O simulador integra o sistema a cada quadro, em subpassos fixos de 2 ms.

**Circuito** — a corrente cai com a força contraeletromotriz e com a resistência de contato:

$$I = \frac{\varepsilon - \text{FCEM}}{R_\text{interna} + R_\text{fio} + R_\text{contato}}, \qquad R_\text{contato} = 0{,}02 + 0{,}14\,\frac{1-q}{q}$$

**Torque e FCEM** — mesma constante de motor $k_t$ nos dois, como manda o SI:

$$\tau = k_t I, \qquad \text{FCEM} = k_t \omega, \qquad k_t = 2 \, (f_r B) \, \ell \, r$$

**Rotação** — dinâmica real, com inércia e três regimes de atrito:

$$J \frac{d\omega}{dt} = \tau - \operatorname{sgn}(\omega)\left(\tau_c + b\,|\omega| + c\,\omega^2\right)$$

Se $\tau \le \tau_c$ o rotor fica **travado**, mesmo com corrente passando — o que acontece de verdade com ímã fraco ou contato ruim.

**Temperatura** — o calor da resistência interna fica na pilha; o fio só sente o que é dissipado nele e nos contatos, e girar aumenta a convecção:

$$C \frac{dT}{dt} = I^2 (R_\text{fio} + R_\text{contato}) - \big(k_0 + k_1|\omega|\big)(T - T_\text{amb})$$

### Constantes

| Parâmetro | Valor | Origem |
|---|---|---|
| $R_\text{interna}$ | 0,25 Ω | pilha AA alcalina típica |
| $I_\text{máx}$ | 6 A | teto prático de curto-circuito de uma AA |
| $f_r$ (fração radial de $B$) | 0,28 | componente radial na posição do fio |
| $\ell$ | 12 mm | trecho de fio dentro do campo, por ramo |
| $r$ | 14 mm | raio médio do laço (ímã de 24 mm) |
| $J$ | 4,0 × 10⁻⁷ kg·m² | laço de cobre de ~2 g a 14 mm do eixo |
| $\tau_c$ | 2,0 × 10⁻⁵ N·m | atrito seco no apoio da ponta sobre o terminal |
| $C$ | 0,77 J/K | ~2 g de cobre ($c$ = 385 J/kg·K) |

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
