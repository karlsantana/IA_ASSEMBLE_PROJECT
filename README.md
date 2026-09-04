# AssemblyGuard — Monitoramento de montagem do Bloq Volt com TinyML

Sistema embarcado que acompanha, **em tempo real e sem nuvem**, as etapas de
montagem do Bloq Volt: um **XIAO ESP32S3 Sense** roda um modelo de detecção
de objetos (**FOMO**, treinado no Edge Impulse) sobre o vídeo da câmera,
cruza a posição dos objetos (`peca`, `pilha`, `ferro_solda`,
`aplicador_cola`) com zonas da bancada e deduz a etapa do ciclo — gerando
alerta quando a sequência sai da ordem e registrando tudo em CSV no microSD.

Projeto do curso **IESTI01 — TinyML** (mentor: Prof. Marcelo Rovai),
desenvolvido na Bottomup Engenharia.

## Como navegar

| Pasta/arquivo | Conteúdo |
|---|---|
| `LEIA-ME_PRIMEIRO.md` | **Comece por aqui** — histórico, decisões, o que está feito e o que falta |
| `docs/` | Plano da V2 (arquitetura e critérios), guias passo a passo (Edge Impulse, anotação) e documentação da V1 |
| `AssemblyGuard_XIAO/` | Firmware (sketch Arduino): FOMO + zonas + máquina de estados + microSD + painel web com overlay |
| `modelos/` | Modelos treinados exportados do Edge Impulse (bibliotecas Arduino, int8 + EON): v1 e v2 |
| `anotar/` | Dataset: 200 frames 320×240 (140 treino = sessão C / 60 teste = sessão B, split por operador/dia) |
| `videos/` | Os 3 vídeos-fonte da câmera Hikvision |
| `scripts/` | Pipelines Python (extração/limpeza de frames, inspeção dos vídeos) e app de inferência no PC |
| `evidencias/` | Folhas de contato e comparativos que embasaram as decisões |
| `historico_v1/` | Artefatos da V1 (classificação de cena) — mantidos como baseline comparativo |

## Estado atual (resumo)

- **Modelo v1 (baseline, anotação 100% automática):** F1 0,54 validação /
  0,23 no teste cross-session — diagnóstico: objetos sem caixa no dataset.
- **Modelo v2:** retreinado após revisão humana das anotações, com a 4ª
  classe `aplicador_cola`.
- **No hardware:** rodando no XIAO — **~143 ms de inferência (~4 fps)**,
  RAM com folga, rede WiFi própria com painel de monitoramento ao vivo
  (vídeo + zonas + detecções + estado).

## Executar o demo

1. Gravar o firmware: Arduino IDE + core **esp32 2.0.17** (a série 3.x é
   incompatível com o export do Edge Impulse), board **XIAO_ESP32S3**,
   **PSRAM: OPI PSRAM**, biblioteca = zip em `modelos/` (Add .ZIP Library).
   Passo a passo completo: `docs/GUIA_EDGE_IMPULSE.md`.
2. Conectar na rede WiFi **"AssemblyGuard"** (senha `bloqvolt123`) e abrir
   **http://192.168.4.1** — vídeo ao vivo com zonas e detecções desenhadas.
3. Demo de mesa: reproduzir `videos/` (sessão C, `15.43.32`) em tela cheia
   num monitor e apontar o XIAO — o ciclo fecha com `== CICLO N COMPLETO`
   no Serial (115200).

## Dataset e treino

O projeto vive no Edge Impulse Studio (projeto **AssemblyGuard**), com o
split **por sessão e por operador** preservado (140/60) — nunca usar o
"Perform train/test split" do Studio. Regras de anotação e fluxo de
retreino: `docs/GUIA_ESTAGIARIA.md`.
