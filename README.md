# Planar Reflection and Transparency via Multi-Pass Rendering in WebGL 2
Esse projeto visa simular reflexo e transparência em tempo real num plano de vidro escuro, com renderização em múltiplos passos (render-to-texture), plano de clipping e Fresnel, em WebGL 2 e GLSL.

## Objetivo
O objetivo é criar, com shaders próprios, um material translúcido que também reflete, como um vidro escuro. Ele é aplicado a um plano que divide uma cena 3D. Na cena, uma maçã e uma melancia giram, uma de cada lado do vidro, sobre um chão de piso. As especificações completas estão no [enunciado do trabalho](enunciado-cg-trabalho-reflexo-e-transparencia.pdf). Em resumo:

- os shaders de vértice e de fragmento são próprios, e o reflexo e a transparência dependem do ângulo de visão;
- o reflexo e a transparência saem de framebuffers renderizados a partir da cena, em vários passos, sem cube map;
- a transparência é ajustável por um uniform;
- todos os objetos têm textura, inclusive o vidro;
- um plano de clipping decide o que aparece em cada passo, sem `if` escolhendo objetos;
- há uma câmera flyby para andar livremente pela cena.

## Como executar
As texturas e os modelos precisam ser servidos por HTTP. Aberta direto pelo arquivo (`file://`), a página é bloqueada pelo navegador e passa a usar substitutos gerados.

```bash
git clone <url-do-repositorio> cg-planar-reflection-transparency
cd cg-planar-reflection-transparency
python3 -m http.server 8000
```

Depois abra http://localhost:8000 num navegador com WebGL 2, como Chrome, Firefox ou Edge recentes. O Live Server do VS Code e o servidor embutido do WebStorm também funcionam.

### Controles

| Ação | Tecla / mouse |
|---|---|
| Olhar ao redor | arrastar o mouse (ou as setas) |
| Andar para frente / trás | W / S |
| Andar para os lados | A / D |
| Descer / subir | Q / E |
| Mover mais rápido | Shift |
| Mostrar / esconder o painel | H |

No painel dá para ajustar a transparência, a refletividade F0, o escurecimento e o ângulo do vidro. Também dá para mostrar no vidro só a textura de um dos passos.

## Como funciona
Cada quadro é renderizado em três passos, no mesmo esquema do shadow map:

| Passo | Destino | Câmera | Plano de clipping |
|---|---|---|---|
| 1. Reflexo | framebuffer | espelhada pelo plano do vidro (`P · V · R`) | mantém só o lado da câmera |
| 2. Transparência | framebuffer | real (`P · V`) | invertido: mantém só o que está atrás do vidro |
| 3. Final | tela | real (`P · V`) | nenhum |

`R` é a matriz de reflexão em relação ao plano do vidro `ax + by + cz + d = 0`:

```
R = | 1-2a²  -2ab   -2ac  -2ad |
    | -2ab   1-2b²  -2bc  -2bd |
    | -2ac   -2bc   1-2c² -2cd |
    |  0      0      0     1   |
```

São dois programas de shader:

- Cena (`vsScene` / `fsScene`): faz a iluminação de Phong (ambiente, difusa e especular, sem sombras), aplica a textura e o plano de clipping. É usado nos três passos.
- Vidro (`vsGlass` / `fsGlass`): junta as texturas dos passos 1 e 2 com a textura do próprio vidro. O peso do reflexo vem do ângulo de visão, pela aproximação de Schlick para Fresnel, `F = F0 + (1 − F0)(1 − cos θ)^5`. O uniform `transparency` controla o quanto se vê através do vidro.

Os modelos `.obj` são lidos por um leitor próprio, sem bibliotecas 3D.

## Estrutura

```
index.html          shaders, leitor de OBJ, câmera e renderização
Common/             MV.js e initShaders.js (bibliotecas de apoio da disciplina)
assets/
  apple/            modelo .obj e textura da maçã
  watermelon/       modelo .obj e textura da melancia
  metal/            textura do vidro
  floor/            textura do chão
enunciado-cg-trabalho-reflexo-e-transparencia.pdf   especificação do trabalho
```

## Créditos
Este projeto foi desenvolvido para a disciplina de Computação Gráfica do curso de Bacharelado em Ciência da Computação do Instituto Federal de Educação, Ciência e Tecnologia de Goiás - Campus Anápolis. Projeto idealizado pelo Prof. Me. [Alessandro Rodrigues e Silva](https://www.escavador.com/sobre/4512567/alessandro-rodrigues-e-silva) e desenvolvido por Karolayne Amábile Brito Borges.

Os modelos 3D e as texturas são provenientes do [ambientCG](https://ambientcg.com), com licença CC0. Foram usados: 3D Apple 001, 3D Melon 001, Metal 049 A e Tiles 143. As bibliotecas de apoio (`MV.js`, `initShaders.js`) são fornecidas na disciplina, mas de autoria original de E. Angel e D. Shreiner, *Interactive Computer Graphics*.