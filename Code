#imports
import time
import random
import copy

# ==========================================
# ESTRUTURAS DE DADOS DO KERNEL
# ==========================================

tabela_processos = []

MAX_PROCESSOS = 5

pid_counter = 1000

# Recursos para Semáforo / Deadlock
recursos = {
    "A": None,
    "B": None
}

# Memória compartilhada (IPC)
memoria_compartilhada = {}


class PCB:
    """Bloco Descritor de Processo"""

    def __init__(self, nome, prioridade=None):

        global pid_counter

        self.pid = pid_counter
        self.nome = nome

        self.estado = "PRONTO"

        self.ciclos_restantes = random.randint(2, 6)

        # 1 = alta | 2 = média | 3 = baixa
        if prioridade is None:
            self.prioridade = random.randint(1, 3)
        else:
            self.prioridade = prioridade

        self.recurso = None

        pid_counter += 1


# ==========================================
# BOOT
# ==========================================

def boot():

    print("Iniciando PyOS Kernel v2.0...")
    time.sleep(1)

    print("Carregando módulos de memória [OK]")
    time.sleep(0.5)

    print("Iniciando escalonador [OK]")
    time.sleep(0.5)

    print("Iniciando sistema de IPC [OK]")
    time.sleep(0.5)

    print("\nBem-vindo ao PyOS.")
    print("Digite 'help' para comandos.\n")


# ==========================================
# PROCESSOS
# ==========================================

def spawn_process(nome):

    if len(tabela_processos) >= MAX_PROCESSOS:

        print("[Kernel] ERRO: Out of Memory!")
        return

    novo = PCB(nome)

    tabela_processos.append(novo)

    print(f"[Kernel] Processo '{nome}' criado com PID {novo.pid}")


# ==========================================
# ESCALONADOR
# ==========================================

def escalonador_tick():

    prontos = [p for p in tabela_processos if p.estado == "PRONTO"]

    if not prontos:

        print("[CPU] Nenhum processo pronto.")
        return

    # PRIORIDADE
    prontos.sort(key=lambda p: p.prioridade)

    processo_atual = prontos[0]

    processo_atual.estado = "EXECUTANDO"

    print(
        f"\n[CPU] Executando PID {processo_atual.pid} "
        f"({processo_atual.nome}) "
        f"[PRIORIDADE {processo_atual.prioridade}]"
    )

    time.sleep(1)

    processo_atual.ciclos_restantes -= 1

    if processo_atual.ciclos_restantes <= 0:

        processo_atual.estado = "ZUMBI"

        print(f"[Kernel] Processo PID {processo_atual.pid} virou ZUMBI.")

    else:

        processo_atual.estado = "PRONTO"

        tabela_processos.remove(processo_atual)

        tabela_processos.append(processo_atual)

        print(f"[Kernel] Chaveamento de contexto do PID {processo_atual.pid}")


# ==========================================
# SEMÁFORO
# ==========================================

def lock_recurso(pid, recurso):

    processo = None

    for p in tabela_processos:

        if p.pid == pid:
            processo = p
            break

    if not processo:

        print("PID não encontrado.")
        return

    if recurso not in recursos:

        print("Recurso inválido.")
        return

    if recursos[recurso] is None:

        recursos[recurso] = pid

        processo.recurso = recurso

        print(f"[Semáforo] PID {pid} bloqueou recurso {recurso}")

    else:

        processo.estado = "BLOQUEADO"

        print(
            f"[Semáforo] Recurso {recurso} ocupado. "
            f"PID {pid} ficou BLOQUEADO."
        )


def unlock_recurso(pid):

    for recurso, dono in recursos.items():

        if dono == pid:

            recursos[recurso] = None

            for p in tabela_processos:

                if p.pid == pid:
                    p.recurso = None

            print(f"[Semáforo] PID {pid} liberou recurso {recurso}")

            return

    print("Esse PID não possui recurso.")


# ==========================================
# DEADLOCK
# ==========================================

def detectar_deadlock():

    dono_a = recursos["A"]
    dono_b = recursos["B"]

    if dono_a and dono_b:

        proc_a = next((p for p in tabela_processos if p.pid == dono_a), None)
        proc_b = next((p for p in tabela_processos if p.pid == dono_b), None)

        if proc_a and proc_b:

            if proc_a.estado == "BLOQUEADO" and proc_b.estado == "BLOQUEADO":

                print("\n[DEADLOCK DETECTADO]")
                print("Os processos entraram em espera circular.\n")
                return

    print("Nenhum deadlock detectado.")


# ==========================================
# IPC
# ==========================================

def escrever_memoria(pid, mensagem):

    memoria_compartilhada[pid] = mensagem

    print(f"[IPC] PID {pid} escreveu mensagem.")


def ler_memoria(pid):

    if pid in memoria_compartilhada:

        print(f"[IPC] Mensagem: {memoria_compartilhada[pid]}")

    else:

        print("Nenhuma mensagem encontrada.")


# ==========================================
# SHELL
# ==========================================

def shell():

    global tabela_processos
    global pid_counter

    while True:

        try:

            comando = input("root@pyos:~# ").strip().split()

            if not comando:
                continue

            acao = comando[0].lower()

            # ======================================
            # HELP
            # ======================================

            if acao == "help":

                print("\n======= COMANDOS =======\n")

                print("spawn [nome]")
                print("ps")
                print("cpu")
                print("run")
                print("block [PID]")
                print("unblock [PID]")
                print("kill [PID]")
                print("wait [PID]")
                print("fork [PID]")

                print("\n--- Semáforos ---")
                print("lock [PID] [A/B]")
                print("unlock [PID]")

                print("\n--- IPC ---")
                print("write [PID] [mensagem]")
                print("read [PID]")

                print("\n--- Outros ---")
                print("deadlock")
                print("clear")
                print("exit\n")

            # ======================================
            # SPAWN
            # ======================================

            elif acao == "spawn":

                if len(comando) > 1:

                    spawn_process(comando[1])

                else:

                    print("Uso correto: spawn [nome]")

            # ======================================
            # PS
            # ======================================

            elif acao == "ps":

                print(
                    f"\n{'PID':<6} | {'NOME':<10} | "
                    f"{'ESTADO':<12} | {'PRIO':<5} | "
                    f"{'CICLOS':<6} | {'RECURSO'}"
                )

                print("-" * 70)

                for p in tabela_processos:

                    print(
                        f"{p.pid:<6} | "
                        f"{p.nome:<10} | "
                        f"{p.estado:<12} | "
                        f"{p.prioridade:<5} | "
                        f"{p.ciclos_restantes:<6} | "
                        f"{str(p.recurso)}"
                    )

                if not tabela_processos:

                    print("Nenhum processo ativo.")

            # ======================================
            # CPU
            # ======================================

            elif acao == "cpu":

                escalonador_tick()

            # ======================================
            # RUN
            # ======================================

            elif acao == "run":

                print("\n[Kernel] Execução automática iniciada...\n")

                while True:

                    prontos = [
                        p for p in tabela_processos
                        if p.estado == "PRONTO"
                    ]

                    if not prontos:
                        break

                    escalonador_tick()

                    time.sleep(0.5)

                print("\n[Kernel] Execução encerrada.")

            # ======================================
            # BLOCK
            # ======================================

            elif acao == "block":

                if len(comando) > 1:

                    pid = int(comando[1])

                    for p in tabela_processos:

                        if p.pid == pid:

                            p.estado = "BLOQUEADO"

                            print(f"PID {pid} bloqueado.")

                            break

                else:

                    print("Uso correto: block [PID]")

            # ======================================
            # UNBLOCK
            # ======================================

            elif acao == "unblock":

                if len(comando) > 1:

                    pid = int(comando[1])

                    for p in tabela_processos:

                        if p.pid == pid:

                            p.estado = "PRONTO"

                            print(f"PID {pid} desbloqueado.")

                            break

                else:

                    print("Uso correto: unblock [PID]")

            # ======================================
            # KILL
            # ======================================

            elif acao == "kill":

                if len(comando) > 1:

                    pid = int(comando[1])

                    antes = len(tabela_processos)

                    tabela_processos = [
                        p for p in tabela_processos
                        if p.pid != pid
                    ]

                    if len(tabela_processos) < antes:

                        print(f"PID {pid} destruído.")

                    else:

                        print("PID não encontrado.")

                else:

                    print("Uso correto: kill [PID]")

            # ======================================
            # WAIT (LIMPA ZUMBI)
            # ======================================

            elif acao == "wait":

                if len(comando) > 1:

                    pid = int(comando[1])

                    for p in tabela_processos:

                        if p.pid == pid and p.estado == "ZUMBI":

                            tabela_processos.remove(p)

                            print(f"PID {pid} removido da RAM.")

                            break

                    else:

                        print("Zumbi não encontrado.")

                else:

                    print("Uso correto: wait [PID]")

            # ======================================
            # FORK
            # ======================================

            elif acao == "fork":

                if len(comando) > 1:

                    pid = int(comando[1])

                    pai = next(
                        (p for p in tabela_processos if p.pid == pid),
                        None
                    )

                    if pai:

                        filho = copy.deepcopy(pai)

                        filho.pid = pid_counter

                        pid_counter += 1

                        filho.nome += "_clone"

                        tabela_processos.append(filho)

                        print(
                            f"[fork] Processo clonado. "
                            f"Novo PID: {filho.pid}"
                        )

                    else:

                        print("PID não encontrado.")

                else:

                    print("Uso correto: fork [PID]")

            # ======================================
            # LOCK
            # ======================================

            elif acao == "lock":

                if len(comando) > 2:

                    pid = int(comando[1])

                    recurso = comando[2].upper()

                    lock_recurso(pid, recurso)

                else:

                    print("Uso correto: lock [PID] [A/B]")

            # ======================================
            # UNLOCK
            # ======================================

            elif acao == "unlock":

                if len(comando) > 1:

                    pid = int(comando[1])

                    unlock_recurso(pid)

                else:

                    print("Uso correto: unlock [PID]")

            # ======================================
            # DEADLOCK
            # ======================================

            elif acao == "deadlock":

                detectar_deadlock()

            # ======================================
            # IPC WRITE
            # ======================================

            elif acao == "write":

                if len(comando) > 2:

                    pid = int(comando[1])

                    mensagem = " ".join(comando[2:])

                    escrever_memoria(pid, mensagem)

                else:

                    print("Uso correto: write [PID] [mensagem]")

            # ======================================
            # IPC READ
            # ======================================

            elif acao == "read":

                if len(comando) > 1:

                    pid = int(comando[1])

                    ler_memoria(pid)

                else:

                    print("Uso correto: read [PID]")

            # ======================================
            # CLEAR
            # ======================================

            elif acao == "clear":

                print("\033[H\033[J", end="")

            # ======================================
            # EXIT
            # ======================================

            elif acao == "exit":

                print("Desligando sistema...")
                break

            # ======================================
            # INVÁLIDO
            # ======================================

            else:

                print(f"bash: {acao}: comando não encontrado.")

        except KeyboardInterrupt:

            print("\nUse 'exit' para sair.")

        except Exception as erro:

            print(f"Erro: {erro}")
if __name__ == "__main__":

    boot()

    shell()
