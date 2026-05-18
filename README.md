# Projeto-de-logica-de-programa-o-em-Python
Desenvolvo projetos de lógica de programação em Python, focados em automação, organização de sistemas e resolução de problemas. Entre eles, um sistema completo de posto de gasolina com controle de estoque, clientes, abastecimento, caixa, relatórios e persistência de dados, utilizando POO e boas práticas.
# ============================================================
# SISTEMA COMPLETO DE POSTO DE GASOLINA
# Linguagem: Python 3
# Recursos:
# - Cadastro de combustíveis
# - Controle de estoque
# - Bombas de combustível
# - Abastecimento
# - Caixa
# - Funcionários
# - Clientes
# - Relatórios
# - Persistência em JSON
# - Logs
# - Validações
# - Interface em terminal
# ============================================================

import json
import os
from datetime import datetime
from dataclasses import dataclass, asdict
from typing import Dict, List

# ============================================================
# CONFIGURAÇÕES
# ============================================================

ARQUIVO_DADOS = "posto_dados.json"
ARQUIVO_LOG = "posto_logs.txt"

# ============================================================
# UTILIDADES
# ============================================================

def log(mensagem):
    with open(ARQUIVO_LOG, "a", encoding="utf-8") as f:
        f.write(f"[{datetime.now()}] {mensagem}\n")


def limpar_tela():
    os.system("cls" if os.name == "nt" else "clear")


def salvar_dados(dados):
    with open(ARQUIVO_DADOS, "w", encoding="utf-8") as f:
        json.dump(dados, f, indent=4, ensure_ascii=False)


def carregar_dados():
    if not os.path.exists(ARQUIVO_DADOS):
        return {
            "combustiveis": {},
            "clientes": {},
            "funcionarios": {},
            "abastecimentos": [],
            "caixa": {
                "saldo": 0
            }
        }

    with open(ARQUIVO_DADOS, "r", encoding="utf-8") as f:
        return json.load(f)

# ============================================================
# MODELOS
# ============================================================

@dataclass
class Combustivel:
    nome: str
    preco_litro: float
    estoque_litros: float


@dataclass
class Cliente:
    nome: str
    cpf: str
    placa: str


@dataclass
class Funcionario:
    nome: str
    cargo: str
    salario: float

# ============================================================
# CLASSE PRINCIPAL
# ============================================================

class PostoGasolina:

    def __init__(self):
        self.dados = carregar_dados()

    # ========================================================
    # COMBUSTÍVEIS
    # ========================================================

    def cadastrar_combustivel(self):
        print("\n=== CADASTRAR COMBUSTÍVEL ===")

        nome = input("Nome: ").strip()

        if nome in self.dados["combustiveis"]:
            print("Combustível já existe.")
            return

        try:
            preco = float(input("Preço por litro: "))
            estoque = float(input("Quantidade em estoque: "))

            combustivel = Combustivel(nome, preco, estoque)

            self.dados["combustiveis"][nome] = asdict(combustivel)

            salvar_dados(self.dados)

            log(f"Combustível cadastrado: {nome}")

            print("Combustível cadastrado com sucesso!")

        except ValueError:
            print("Valor inválido.")

    def listar_combustiveis(self):
        print("\n=== COMBUSTÍVEIS ===")

        if not self.dados["combustiveis"]:
            print("Nenhum combustível cadastrado.")
            return

        for nome, dados in self.dados["combustiveis"].items():
            print(f"""
Nome: {nome}
Preço/Litro: R$ {dados['preco_litro']:.2f}
Estoque: {dados['estoque_litros']} L
-----------------------------
""")

    def atualizar_preco(self):
        print("\n=== ATUALIZAR PREÇO ===")

        nome = input("Combustível: ").strip()

        if nome not in self.dados["combustiveis"]:
            print("Combustível não encontrado.")
            return

        try:
            novo_preco = float(input("Novo preço: "))

            self.dados["combustiveis"][nome]["preco_litro"] = novo_preco

            salvar_dados(self.dados)

            log(f"Preço atualizado: {nome} -> {novo_preco}")

            print("Preço atualizado!")

        except ValueError:
            print("Valor inválido.")

    # ========================================================
    # CLIENTES
    # ========================================================

    def cadastrar_cliente(self):
        print("\n=== CADASTRAR CLIENTE ===")

        cpf = input("CPF: ").strip()

        if cpf in self.dados["clientes"]:
            print("Cliente já cadastrado.")
            return

        nome = input("Nome: ").strip()
        placa = input("Placa do veículo: ").strip()

        cliente = Cliente(nome, cpf, placa)

        self.dados["clientes"][cpf] = asdict(cliente)

        salvar_dados(self.dados)

        log(f"Cliente cadastrado: {nome}")

        print("Cliente cadastrado com sucesso!")

    def listar_clientes(self):
        print("\n=== CLIENTES ===")

        if not self.dados["clientes"]:
            print("Nenhum cliente.")
            return

        for cpf, c in self.dados["clientes"].items():
            print(f"""
Nome: {c['nome']}
CPF: {cpf}
Placa: {c['placa']}
---------------------
""")

    # ========================================================
    # FUNCIONÁRIOS
    # ========================================================

    def cadastrar_funcionario(self):
        print("\n=== CADASTRAR FUNCIONÁRIO ===")

        nome = input("Nome: ").strip()
        cargo = input("Cargo: ").strip()

        try:
            salario = float(input("Salário: "))

            funcionario = Funcionario(nome, cargo, salario)

            self.dados["funcionarios"][nome] = asdict(funcionario)

            salvar_dados(self.dados)

            log(f"Funcionário cadastrado: {nome}")

            print("Funcionário cadastrado!")

        except ValueError:
            print("Valor inválido.")

    def listar_funcionarios(self):
        print("\n=== FUNCIONÁRIOS ===")

        if not self.dados["funcionarios"]:
            print("Nenhum funcionário.")
            return

        for nome, f in self.dados["funcionarios"].items():
            print(f"""
Nome: {nome}
Cargo: {f['cargo']}
Salário: R$ {f['salario']:.2f}
--------------------------
""")

    # ========================================================
    # ABASTECIMENTO
    # ========================================================

    def abastecer(self):
        print("\n=== ABASTECIMENTO ===")

        cpf = input("CPF do cliente: ").strip()

        if cpf not in self.dados["clientes"]:
            print("Cliente não encontrado.")
            return

        cliente = self.dados["clientes"][cpf]

        self.listar_combustiveis()

        combustivel_nome = input("Tipo de combustível: ").strip()

        if combustivel_nome not in self.dados["combustiveis"]:
            print("Combustível inválido.")
            return

        combustivel = self.dados["combustiveis"][combustivel_nome]

        try:
            litros = float(input("Quantidade de litros: "))

            if litros <= 0:
                print("Quantidade inválida.")
                return

            if litros > combustivel["estoque_litros"]:
                print("Estoque insuficiente.")
                return

            total = litros * combustivel["preco_litro"]

            combustivel["estoque_litros"] -= litros

            self.dados["caixa"]["saldo"] += total

            registro = {
                "cliente": cliente["nome"],
                "cpf": cpf,
                "combustivel": combustivel_nome,
                "litros": litros,
                "valor_total": total,
                "data": str(datetime.now())
            }

            self.dados["abastecimentos"].append(registro)

            salvar_dados(self.dados)

            log(f"Abastecimento realizado: {registro}")

            print("\n=== RECIBO ===")
            print(f"Cliente: {cliente['nome']}")
            print(f"Combustível: {combustivel_nome}")
            print(f"Litros: {litros}")
            print(f"Total: R$ {total:.2f}")

        except ValueError:
            print("Valor inválido.")

    # ========================================================
    # RELATÓRIOS
    # ========================================================

    def relatorio_caixa(self):
        print("\n=== CAIXA ===")

        saldo = self.dados["caixa"]["saldo"]

        print(f"Saldo total: R$ {saldo:.2f}")

    def relatorio_abastecimentos(self):
        print("\n=== ABASTECIMENTOS ===")

        if not self.dados["abastecimentos"]:
            print("Nenhum abastecimento.")
            return

        for a in self.dados["abastecimentos"]:
            print(f"""
Cliente: {a['cliente']}
CPF: {a['cpf']}
Combustível: {a['combustivel']}
Litros: {a['litros']}
Total: R$ {a['valor_total']:.2f}
Data: {a['data']}
-----------------------------------
""")

    # ========================================================
    # MENU
    # ========================================================

    def menu(self):

        while True:

            print("""
=================================================
         SISTEMA DE POSTO DE GASOLINA
=================================================

1 - Cadastrar combustível
2 - Listar combustíveis
3 - Atualizar preço

4 - Cadastrar cliente
5 - Listar clientes

6 - Cadastrar funcionário
7 - Listar funcionários

8 - Realizar abastecimento

9 - Relatório de caixa
10 - Relatório de abastecimentos

0 - Sair
=================================================
""")

            opcao = input("Escolha: ")

            limpar_tela()

            if opcao == "1":
                self.cadastrar_combustivel()

            elif opcao == "2":
                self.listar_combustiveis()

            elif opcao == "3":
                self.atualizar_preco()

            elif opcao == "4":
                self.cadastrar_cliente()

            elif opcao == "5":
                self.listar_clientes()

            elif opcao == "6":
                self.cadastrar_funcionario()

            elif opcao == "7":
                self.listar_funcionarios()

            elif opcao == "8":
                self.abastecer()

            elif opcao == "9":
                self.relatorio_caixa()

            elif opcao == "10":
                self.relatorio_abastecimentos()

            elif opcao == "0":
                print("Encerrando sistema...")
                break

            else:
                print("Opção inválida.")

            input("\nPressione ENTER para continuar...")
            limpar_tela()

# ============================================================
# EXECUÇÃO
# ============================================================

if __name__ == "__main__":
    sistema = PostoGasolina()
    sistema.menu()
