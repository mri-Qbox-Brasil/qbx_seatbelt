# qbx_seatbelt — Manual

Cinto de segurança e harness de corrida: bloqueia a ejeção pelo para-brisa, toca som de fivela e expõe o estado via statebag para HUDs.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Controles](#controles)
5. [Item harness](#item-harness)
6. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
7. [Localização](#localização)
8. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `qbx_core` | Sim | Notificações (`Notify`), item usável (`CreateUseableItem`), helpers `qbx.loadAudioBank` / `qbx.playAudio` |
| `ox_lib` | Sim | Locale, `lib.addKeybind`, `lib.progressCircle`, `cache` |
| `ox_inventory` | Sim | Slot e metadata (`durability`) do item `harness` |

---

## Instalação

1. Copie a pasta `qbx_seatbelt` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure qbx_seatbelt
   ```
3. Cadastre o item `harness` no `ox_inventory` (`data/items.lua`). O item precisa ser usável e suporta a metadata `durability` (cada uso consome 5 pontos; ao chegar em 0 o item é removido).
4. O recurso força a convar `game_enableFlyThroughWindscreen` para `true` no start do servidor (`SetConvarReplicated`). Não é necessário defini-la manualmente.

---

## Configuração

Arquivo: `config/client.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `keybind` | string | Sim | Tecla padrão para alternar o cinto. Padrão: `G`. O jogador pode remapear pelas configurações do FiveM |
| `useMPH` | bool | Sim | `true` interpreta as velocidades abaixo em MPH; `false` (padrão) em KM/H |
| `minSpeedUnbuckled` | number | Sim | Velocidade mínima para ser ejetado pelo para-brisa com o cinto solto. Padrão: `20.0` |
| `minSpeedBuckled` | number | Sim | Velocidade mínima para ser ejetado pelo para-brisa com o cinto afivelado. Padrão: `160.0` |
| `harness.disableFlyingThroughWindscreen` | bool | Sim | `true` desativa completamente a ejeção pelo para-brisa enquanto o harness está equipado (ped config flag 32) |
| `harness.minSpeed` | number | Sim | Velocidade mínima para ejeção com harness equipado. Só é usada quando `harness.disableFlyingThroughWindscreen = false`. Padrão: `200.0` |

---

## Controles

| Ação | Tecla | Condição |
|---|---|---|
| Alternar cinto | `G` (keybind `mri_Qtoggleseatbelt`) | Precisa estar dentro de um veículo, com o menu de pausa fechado, e o veículo não pode ser das classes 8 (moto), 13 (bicicleta) ou 14 (barco) |

Com cinto ou harness ativos, o controle 75 (sair do veículo) fica bloqueado — é preciso desafivelar antes de sair.

---

## Item harness

Usar o item `harness` no inventário dispara uma barra de progresso de 5 segundos (equipar ou remover). Regras aplicadas pelo código:

- Não é possível equipar o harness com o cinto afivelado.
- Só funciona dentro de um veículo que não seja das classes 8, 13 ou 14.
- A cada equipagem, o servidor desconta 5 pontos de `durability` da metadata do item; quando o valor chega a 0 ou menos, o item é removido do inventário.

---

## Entrypoints para outros recursos

### Statebags

O estado é publicado em `LocalPlayer.state` — é a forma recomendada de um HUD ler o cinto.

```lua
local seatbeltOn = LocalPlayer.state.seatbelt
local harnessOn = LocalPlayer.state.harness
```

Ambos são zerados automaticamente quando o jogador sai do veículo.

### Evento `seatbelt:client:ToggleSeatbelt`

Disparado no cliente sempre que o cinto ou o harness mudam de estado (sem argumentos).

```lua
AddEventHandler('seatbelt:client:ToggleSeatbelt', function()
    -- ler LocalPlayer.state.seatbelt / LocalPlayer.state.harness
end)
```

### Export `HasHarness` (cliente)

Marcado como deprecado no código; prefira `LocalPlayer.state.harness`.

```lua
local hasHarness = exports.qbx_seatbelt:HasHarness()
```

### Eventos internos

| Evento | Lado | Descrição |
|---|---|---|
| `qbx_seatbelt:client:UseHarness` | Cliente | Disparado pelo servidor quando o item `harness` é usado |
| `qbx_seatbelt:server:equip` | Servidor | Recebe o slot do item e aplica o desgaste de `durability` |

---

## Localização

Strings via `ox_lib` locale, em `locales/`:

- `en.json` — inglês
- `da.json` — dinamarquês
- `pt-br.json` — português do Brasil

Idioma ativo pela convar:

```
setr ox:locale "pt-br"
```

---

## Estrutura de arquivos

```
qbx_seatbelt/
├── client/
│   └── main.lua                        — cinto, harness, keybind, som de fivela, statebags
├── server/
│   └── main.lua                        — item usável `harness`, desgaste de durability, convar do para-brisa
├── config/
│   └── client.lua                      — keybind, unidade de velocidade e limiares de ejeção
├── audiodirectory/
│   ├── seatbelt_sounds.awc             — wavepack com os sons de afivelar/desafivelar
│   ├── seatbelt_sounds.awc.xml
│   └── seatbelt_sounds/                — WAVs de origem (carbuckle_left, carunbuckle_left)
├── data/
│   ├── seatbelt_sounds.dat54.rel       — sounddata do soundset `seatbelt_soundset`
│   └── seatbelt_sounds.dat54.rel.xml
├── locales/
│   ├── da.json
│   ├── en.json
│   └── pt-br.json
└── fxmanifest.lua
```
