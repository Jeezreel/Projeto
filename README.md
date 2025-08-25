# PROJETO INTEGRADO INOVAÇÃO - ANÁLISE E DESENVOLVIMENTO DE SISTEMAS

"""
SISTEMA CLINICA VIDA+
Passo 2 do Projeto Integrado Inovacao - ADS
Autor: Jeezreel Guilherme da Silva Santos
Descrição: Cadastro de pacientes, estatísticas, busca e listagem.
Requisitos: usar listas e dicionários, menu em loop, tratamento de erros.
"""

from typing import List, Dict, Optional, Tuple

Paciente = Dict[str, object]  # {"nome": str, "idade": int, "telefone": str}


# ------------------------- Utilitários de Entrada ------------------------- #
def ler_inteiro(mensagem: str, minimo: Optional[int] = None, maximo: Optional[int] = None) -> int:
    """
    Lê um inteiro do usuário com validação opcional de intervalo.
    """
    while True:
        valor = input(mensagem).strip()
        try:
            numero = int(valor)
            if minimo is not None and numero < minimo:
                print(f"Valor deve ser >= {minimo}. Tente novamente.")
                continue
            if maximo is not None and numero > maximo:
                print(f"Valor deve ser <= {maximo}. Tente novamente.")
                continue
            return numero
        except ValueError:
            print("Entrada inválida. Digite um número inteiro válido.")


def ler_texto_nao_vazio(mensagem: str, max_len: Optional[int] = None) -> str:
    """
    Lê um texto não vazio do usuário, opcionalmente limitando o tamanho.
    """
    while True:
        texto = input(mensagem).strip()
        if not texto:
            print("O campo não pode ficar vazio. Tente novamente.")
            continue
        if max_len is not None and len(texto) > max_len:
            print(f"O texto não pode exceder {max_len} caracteres. Tente novamente.")
            continue
        return texto


def normalizar_nome(nome: str) -> str:
    """
    Normaliza o nome para comparação (casefold + espaços únicos).
    """
    return " ".join(nome.split()).casefold()


# ------------------------- Operações de Pacientes ------------------------- #
def cadastrar_paciente(pacientes: List[Paciente]) -> None:
    """
    Cadastra um paciente com nome, idade e telefone.
    Evita duplicidade exata de nome + telefone.
    """
    print("\n--- Cadastro de Paciente ---")
    nome = ler_texto_nao_vazio("Nome do paciente: ", max_len=100)
    idade = ler_inteiro("Idade: ", minimo=0, maximo=130)
    telefone = ler_texto_nao_vazio("Telefone (ex.: (11) 99999-9999): ", max_len=30)

    # Verificar duplicidade simples (mesmo nome e telefone)
    ja_existe = any(
        normalizar_nome(p["nome"]) == normalizar_nome(nome) and str(p["telefone"]).strip() == telefone.strip()
        for p in pacientes
    )
    if ja_existe:
        print("Atenção: já existe um paciente cadastrado com o mesmo nome e telefone.")
        confirmar = ler_texto_nao_vazio("Deseja cadastrar mesmo assim? (S/N): ").upper()
        if confirmar != "S":
            print("Cadastro cancelado.")
            return

    pacientes.append({"nome": nome, "idade": idade, "telefone": telefone})
    print("Paciente cadastrado com sucesso!")


def calcular_estatisticas(pacientes: List[Paciente]) -> Optional[Tuple[int, float, Paciente, Paciente]]:
    """
    Calcula total, idade média, paciente mais novo e mais velho.
    Retorna None se não houver pacientes.
    """
    if not pacientes:
        return None

    total = len(pacientes)
    soma_idades = sum(int(p["idade"]) for p in pacientes)
    media = soma_idades / total

    mais_novo = min(pacientes, key=lambda p: int(p["idade"]))
    mais_velho = max(pacientes, key=lambda p: int(p["idade"]))
    return total, media, mais_novo, mais_velho


def exibir_estatisticas(pacientes: List[Paciente]) -> None:
    print("\n--- Estatísticas da Clínica ---")
    resultado = calcular_estatisticas(pacientes)
    if resultado is None:
        print("Nenhum paciente cadastrado.")
        return

    total, media, mais_novo, mais_velho = resultado
    print(f"Número total de pacientes: {total}")
    print(f"Idade média dos pacientes: {media:.2f} anos")
    print(f"Paciente mais novo: {mais_novo['nome']} ({mais_novo['idade']} anos)")
    print(f"Paciente mais velho: {mais_velho['nome']} ({mais_velho['idade']} anos)")


def buscar_paciente(pacientes: List[Paciente]) -> None:
    print("\n--- Buscar Paciente ---")
    termo = ler_texto_nao_vazio("Digite o nome (ou parte do nome) para buscar: ")
    termo_norm = normalizar_nome(termo)

    encontrados = [
        p for p in pacientes
        if termo_norm in normalizar_nome(str(p["nome"]))
    ]

    if not encontrados:
        print("Nenhum paciente encontrado.")
        return

    print(f"Foram encontrados {len(encontrados)} paciente(s):")
    listar_pacientes(encontrados, cabecalho=False)


def listar_pacientes(pacientes: List[Paciente], cabecalho: bool = True) -> None:
    if cabecalho:
        print("\n--- Lista de Pacientes ---")
    if not pacientes:
        print("Nenhum paciente cadastrado.")
        return

    # Larguras para tabela simples
    w_nome = max(4, min(40, max(len(str(p["nome"])) for p in pacientes)))
    w_tel = max(8, min(20, max(len(str(p["telefone"])) for p in pacientes)))
    linha = f"{'Nome'.ljust(w_nome)}  {'Idade':>5}  {'Telefone'.ljust(w_tel)}"
    print(linha)
    print("-" * len(linha))

    for p in pacientes:
        print(f"{str(p['nome']).ljust(w_nome)}  {int(p['idade']):>5}  {str(p['telefone']).ljust(w_tel)}")


# ------------------------- Menu do Sistema ------------------------- #
def exibir_menu() -> None:
    print("\n=== SISTEMA CLÍNICA VIDA+ ===")
    print("1. Cadastrar paciente")
    print("2. Ver estatísticas")
    print("3. Buscar paciente")
    print("4. Listar todos os pacientes")
    print("5. Sair")


def loop_principal() -> None:
    pacientes: List[Paciente] = []

    while True:
        exibir_menu()
        opcao = ler_inteiro("Escolha uma opção: ", minimo=1, maximo=5)

        if opcao == 1:
            cadastrar_paciente(pacientes)
        elif opcao == 2:
            exibir_estatisticas(pacientes)
        elif opcao == 3:
            buscar_paciente(pacientes)
        elif opcao == 4:
            listar_pacientes(pacientes)
        elif opcao == 5:
            print("Saindo do sistema. Até logo!")
            break


# ------------------------- Execução ------------------------- #
if __name__ == "__main__":
    loop_principal()
