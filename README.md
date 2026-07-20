# kb2xbox

Converta um teclado em um ou mais controles de Xbox.

## Descrição

No Linux, todo teclado é tratado como o mesmo dispositivo de entrada, o que dificulta
jogar cooperativo local com múltiplos jogadores. O **kb2xbox** lê eventos de um teclado
dedicado (geralmente um segundo ou terceiro teclado) e emula controles de Xbox via
`/dev/uinput`, permitindo que cada jogador use seu próprio teclado como controle.

## Requisitos

- Linux
- Python 3.9+
- [python-libevdev](https://python-libevdev.readthedocs.io/en/latest/) (`pip install libevdev`)
- Acesso de escrita ao `/dev/uinput` (veja [Instalação](#instalação))

## Instalação

### 1. Crie um ambiente virtual e instale as dependências

```bash
python3 -m venv venv
source venv/bin/activate
pip install libevdev
```

### 2. Torne `/dev/uinput` gravável

```bash
sudo chmod 666 /dev/uinput
```

### 3. Encontre seu teclado

```bash
sudo python kb2xbox.py --list
```

Procure pelo teclado desejado na saída e anote o caminho do dispositivo de evento
(ex.: `/dev/input/event5`).

## Como usar

```bash
sudo python kb2xbox.py -d <DISPOSITIVO> <ARQUIVOS_DE_CONFIG...>
```

### Exemplo

```bash
sudo python kb2xbox.py -d /dev/input/event5 config/xbox.cfg config/xbox2.cfg
```

Isso emula 2 controles de Xbox no mesmo teclado físico:

| Controle 1 | Controle 2 |
|---|---|
| Setas direcionais -> Analógico esquerdo | E, S, D, F -> Analógico esquerdo |
| Right Alt -> Botão A | Left Shift -> Botão A |
| Henkan -> Botão B | Caps Lock -> Botão B |
| Katakanahiragana -> Botão X | Tab -> Botão X |

## Atalhos do teclado

| Atalho | Ação |
|---|---|
| `Ctrl+F1` | Alternar captura (libera o teclado para o sistema) |
| `Ctrl+Esc` | Sair |

Enquanto a captura estiver ativa, as teclas são enviadas apenas para os controles
emulados. Pressione `Ctrl+F1` para liberar o teclado quando precisar digitar
normalmente.

## Arquivos de configuração

Os arquivos de config mapeiam teclas físicas do teclado para botões e eixos do
controle de Xbox.

### Formato

```ini
NAME=<nome do controle>
VENDOR=<ID do fabricante em hex>
PRODUCT=<ID do produto em hex>
VERSION=<versão em hex>

# Botões (EV_KEY)
BTN_SOUTH=KEY_<nome>       # A
BTN_EAST=KEY_<nome>        # B
BTN_NORTH=KEY_<nome>       # X
BTN_WEST=KEY_<nome>        # Y
BTN_TL=KEY_<nome>          # Gatilho esquerdo (LB)
BTN_TR=KEY_<nome>          # Gatilho direito (RB)
BTN_SELECT=KEY_<nome>      # Select / Voltar
BTN_START=KEY_<nome>       # Start
BTN_THUMBL=KEY_<nome>      # Clique do analógico esquerdo
BTN_THUMBR=KEY_<nome>      # Clique do analógico direito
BTN_MODE=KEY_<nome>        # Botão Xbox

# Direcional (EV_ABS)
ABS_HAT0X=KEY_<E>,KEY_<D>   # Direcional esquerda, direita
ABS_HAT0Y=KEY_<C>,KEY_<B>   # Direcional cima, baixo

# Analógico esquerdo (EV_ABS)
ABS_X=KEY_<E>,KEY_<D>       # Eixo X do analógico esquerdo
ABS_Y=KEY_<C>,KEY_<B>       # Eixo Y do analógico esquerdo

# Analógico direito (EV_ABS)
ABS_RX=KEY_<E>,KEY_<D>
ABS_RY=KEY_<C>,KEY_<B>

# Gatilhos analógicos (EV_ABS) — suporta divisão em múltiplas teclas
ABS_Z=KEY_<nome>[,KEY_...]  # Gatilho LT
ABS_RZ=KEY_<nome>[,KEY_...] # Gatilho RT
```

- Deixe o valor vazio para não mapear o botão/eixo.
- Linhas começando com `#` são comentários.
- Múltiplas teclas separadas por vírgula em um eixo são divididas em etapas iguais.

### Nomes das teclas

Use os nomes `KEY_*` padrão do Linux input-event-codes (ex.: `KEY_A`, `KEY_LEFT`,
`KEY_SPACE`, `KEY_LEFTSHIFT`, `KEY_HENKAN`).

### Configs inclusos

| Arquivo | Jogo / Finalidade |
|---|---|
| `config/xbox.cfg` | Layout genérico de Xbox |
| `config/xbox2.cfg` | Segundo layout genérico de Xbox |
| `config/Overcooked2-1.cfg` | Overcooked 2 — Jogador 1 |
| `config/Overcooked2-2.cfg` | Overcooked 2 — Jogador 2 |
| `config/Unrailed-1.cfg` | Unrailed! — Jogador 1 |
| `config/Unrailed-2.cfg` | Unrailed! — Jogador 2 |
| `config/TheEscapist2.cfg` | The Escapists 2 |

## Solução de problemas

**"Permission denied" ao acessar `/dev/uinput`**: execute `sudo chmod 666 /dev/uinput`.

**Nenhum teclado encontrado com `--list`**: execute com `sudo` — a leitura de
`/dev/input/event*` requer permissão de root.

**Teclas não funcionam**: verifique se o nome da tecla está correto (use `evtest`)
e se o seu teclado possui essa tecla.

## Rede: usar teclado de outro computador

Encaminhe um teclado remoto via TCP para que ele apareça como um dispositivo local:

**Receptor** (a máquina rodando o kb2xbox):
```bash
nc -l -p 4444 > /dev/input/by-path/platform-i8042-serio-0-event-kbd
```

**Remetente** (o teclado remoto):
```bash
cat /dev/input/by-path/platform-i8042-serio-0-event-kbd | nc <IP> 4444
```
