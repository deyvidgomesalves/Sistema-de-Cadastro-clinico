import sqlite3


conn = sqlite3.connect("consultorio.db")
cursor = conn.cursor()

cursor.execute("CREATE TABLE IF NOT EXISTS pacientes (id INTEGER PRIMARY KEY, nome TEXT, nascimento TEXT, sexo TEXT, cpf TEXT, telefone TEXT, celular TEXT, celular2 TEXT, whatsapp TEXT, cidade TEXT, bairro TEXT, rua_numero TEXT, complemento TEXT)")
cursor.execute("CREATE TABLE IF NOT EXISTS planos_de_saude (id INTEGER PRIMARY KEY, nome TEXT, paciente_id INTEGER, numerocartao TEXT, validade TEXT)")
cursor.execute("CREATE TABLE IF NOT EXISTS consultas (id INTEGER PRIMARY KEY, paciente_id INTEGER, medico_crm TEXT, data TEXT, hora TEXT)")
cursor.execute("CREATE TABLE IF NOT EXISTS medicos (id INTEGER PRIMARY KEY, nome TEXT)")
cursor.execute("CREATE TABLE IF NOT EXISTS fichas (id INTEGER PRIMARY KEY, paciente_id INTEGER, medico_nome TEXT, dia_hora_consulta TEXT, doenca TEXT)")


cursor.execute("PRAGMA table_info(fichas)")
columns = [column[1] for column in cursor.fetchall()]
if "medico_nome" not in columns:
    cursor.execute("ALTER TABLE fichas ADD COLUMN medico_nome TEXT")

conn.commit()

def get_numero_input(mensagem, tamanho_esperado):
    while True:
        entrada = input(mensagem)
        if entrada.isdigit() and len(entrada) == tamanho_esperado:
            return entrada
        print(f"Entrada inválida. Certifique-se de que seja um número com exatamente {tamanho_esperado} dígitos.")


def cadastrar_paciente():
    nome = input("Digite o nome do paciente: ")
    nascimento = get_numero_input("Digite a data de nascimento do paciente (DDMMAAAA): ", 8)
    sexo = input("Digite o sexo do paciente: ")
    cpf = get_numero_input("Digite o CPF do paciente: ",11)
    telefone = get_numero_input("Digite o telefone do paciente: ", 9)
    celular = get_numero_input("Digite o celular do paciente: ", 11)
    celular2 = get_numero_input("Digite o segundo celular do paciente: ", 11)
    whatsapp = get_numero_input("Digite o WhatsApp do paciente: ", 11)
    cidade = input("Digite a cidade do paciente: ")
    bairro = input("Digite o bairro do paciente: ")
    rua_numero = input("Digite a rua e número do paciente: ")
    complemento = input("Digite informações de complemento: ")

    cursor.execute("INSERT INTO pacientes (nome, nascimento, sexo, cpf, telefone, celular, celular2, whatsapp, cidade, bairro, rua_numero, complemento) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)",
                   (nome, nascimento, sexo, cpf, telefone, celular, celular2, whatsapp, cidade, bairro, rua_numero, complemento))
    conn.commit()
    print("Cadastro com sucesso.")


def deletar_paciente():
    nome = input("Digite o nome do paciente a ser deletado: ")
    cursor.execute("DELETE FROM pacientes WHERE nome=?", (nome,))
    cursor.execute("DELETE FROM planos_de_saude WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (nome,))
    cursor.execute("DELETE FROM consultas WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (nome,))
    conn.commit()


def consultar_paciente():
    nome = input("Digite o nome do paciente a ser consultado: ")
    cursor.execute("SELECT * FROM pacientes WHERE nome=?", (nome,))
    paciente = cursor.fetchone()
    if paciente:
        print("Dados do paciente:")
        print(f"Nome: {paciente[1]}")
        print(f"Nascimento: {paciente[2]}")
        print(f"Sexo: {paciente[3]}")
        print(f"CPF: {paciente[4]}")
        print(f"Telefone: {paciente[5]}")
        print(f"Celular: {paciente[6]}")
        print(f"Celular2: {paciente[7]}")
        print(f"WhatsApp: {paciente[8]}")
        print(f"Cidade: {paciente[9]}")
        print(f"Bairro: {paciente[10]}")
        print(f"Rua/Número: {paciente[11]}")
        print(f"Complemento: {paciente[12]}")
    else:
        print("Paciente não encontrado.")


def atualizar_paciente():
    nome = input("Digite o nome do paciente a ser atualizado: ")
    cursor.execute("SELECT * FROM pacientes WHERE nome=?", (nome,))
    paciente = cursor.fetchone()
    if paciente:
        novo_nome = input("Novo nome do paciente: ")
        novo_nascimento = get_numero_input("Novo nascimento do paciente (DDMMAAAA): ", 8)
        novo_sexo = input("Novo sexo do paciente: ")
        novo_cpf = get_numero_input("Novo CPF do paciente: ", 11)
        novo_telefone = get_numero_input("Novo telefone do paciente: ", 9)
        novo_celular = get_numero_input("Novo celular do paciente: ", 11)
        novo_celular2 = get_numero_input("Novo segundo celular do paciente: ", 11)
        novo_whatsapp = get_numero_input("Novo WhatsApp do paciente: ", 11)
        nova_cidade = input("Nova cidade do paciente: ")
        novo_bairro = input("Novo bairro do paciente: ")
        nova_rua_numero = input("Nova rua e número do paciente: ")
        novo_complemento = input("Novas informações de complemento: ")

        cursor.execute("UPDATE pacientes SET nome=?, nascimento=?, sexo=?, cpf=?, telefone=?, celular=?, celular2=?, whatsapp=?, cidade=?, bairro=?, rua_numero=?, complemento=? WHERE nome=?", (novo_nome, novo_nascimento, novo_sexo, novo_cpf, novo_telefone, novo_celular, novo_celular2, novo_whatsapp, nova_cidade, novo_bairro, nova_rua_numero, novo_complemento, nome))
        conn.commit()
        print("Dados do paciente atualizados com sucesso.")
    else:
        print("Paciente não encontrado.")


def listar_pacientes():
    cursor.execute("SELECT nome FROM pacientes")
    pacientes = cursor.fetchall()
    if pacientes:
        print("\nLista de Pacientes:")
        for paciente in pacientes:
            print(paciente[0])
    else:
        print("Nenhum paciente cadastrado.")


def cadastrar_plano_de_saude():
    nome_plano = input("Digite o nome do plano de saúde: ")
    numerocartao = int(input("Digite o número do cartão do plano de saúde: "))
    validade = int(input("Digite a validade do plano de saúde: "))
    paciente = input("Digite o nome do paciente a ser vinculado ao plano: ")
    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (paciente,))
    paciente_id = cursor.fetchone()
    if paciente_id:
        cursor.execute("INSERT INTO planos_de_saude (nome, paciente_id, numerocartao, validade) VALUES (?, ?, ?, ?)", (nome_plano, paciente_id[0], numerocartao, validade))
        conn.commit()
        print("Dados cadastrado com sucesso.")
    else:
        print("Paciente não encontrado.")


def pesquisar_plano_de_saude():
    paciente = input("Digite o nome do paciente para pesquisar o plano de saúde: ")
    cursor.execute(
        "SELECT planos_de_saude.nome, planos_de_saude.numerocartao, planos_de_saude.validade FROM planos_de_saude JOIN pacientes ON planos_de_saude.paciente_id = pacientes.id WHERE pacientes.nome=?",
        (paciente,))
    planos = cursor.fetchall()
    if planos:
        print(f"Planos de Saúde de {paciente}:")
        for plano in planos:
            print(f"Nome do Plano: {plano[0]}")
            print(f"Número do Cartão: {plano[1]}")
            print(f"Validade: {plano[2]}")
    else:
        print(f"Nenhum plano de saúde encontrado para {paciente}.")


def listar_planos_de_saude():
    cursor.execute("SELECT nome, numerocartao, validade FROM planos_de_saude")
    planos = cursor.fetchall()
    if planos:
        print("\nLista de Planos de Saúde:")
        for plano in planos:
            print(f"Nome do Plano: {plano[0]}")
            print(f"Número do Cartão: {plano[1]}")
            print(f"Validade: {plano[2]}")
    else:
        print("Nenhum plano de saúde cadastrado.")


def atualizar_plano_de_saude():
    nome_paciente = input("Digite o nome do paciente para atualizar o plano de saúde: ")
    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (nome_paciente,))
    paciente_id = cursor.fetchone()
    if paciente_id:
        cursor.execute("SELECT id, nome FROM planos_de_saude WHERE paciente_id=?", (paciente_id[0],))
        planos = cursor.fetchall()
        if planos:
            print(f"Planos de Saúde do paciente '{nome_paciente}':")
            for plano in planos:
                print(f"{plano[0]}. {plano[1]}")
            plano_id = int(input("Digite o número do plano de saúde que deseja atualizar: "))
            novo_nome_plano = input("Novo nome do plano de saúde: ")

            cursor.execute("UPDATE planos_de_saude SET nome=? WHERE id=?", (novo_nome_plano, plano_id))
            conn.commit()
            print(f"Plano de saúde atualizado com sucesso para '{novo_nome_plano}'.")
        else:
            print(f"O paciente '{nome_paciente}' não tem planos de saúde cadastrados.")
    else:
        print(f"Paciente '{nome_paciente}' não encontrado.")


def cancelar_plano_de_saude():
    nome_paciente = input("Digite o nome do paciente para cancelar o plano de saúde: ")
    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (nome_paciente,))
    paciente_id = cursor.fetchone()
    if paciente_id:
        cursor.execute("SELECT id, nome FROM planos_de_saude WHERE paciente_id=?", (paciente_id[0],))
        planos = cursor.fetchall()
        if planos:
            print(f"Planos de Saúde do paciente '{nome_paciente}':")
            for plano in planos:
                print(f"{plano[0]}. {plano[1]}")
            plano_id = int(input("Digite o número do plano de saúde que deseja cancelar: "))

            cursor.execute("DELETE FROM planos_de_saude WHERE id=?", (plano_id,))
            conn.commit()
            print(f"Plano de saúde cancelado com sucesso.")
        else:
            print(f"O paciente '{nome_paciente}' não tem planos de saúde cadastrados.")
    else:
        print(f"Paciente '{nome_paciente}' não encontrado.")
def cadastrar_consulta():
    paciente = input("Digite o nome do paciente: ")
    medico_crm = int(input("Digite o CRM do médico: "))
    data = int(input("Digite a data da consulta (DDMMAAAA): "))
    hora = int(input("Digite a hora da consulta (HH:MM): "))
    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (paciente,))
    paciente_id = cursor.fetchone()
    if paciente_id:
        cursor.execute("INSERT INTO consultas (paciente_id, medico_crm, data, hora) VALUES (?, ?, ?, ?)", (paciente_id[0], medico_crm, data, hora))
        conn.commit()
        print("Dados cadastrado com sucesso.")
    else:
        print("Paciente não encontrado.")


def remarcar_consulta():
    paciente = input("Digite o nome do paciente: ")


    cursor.execute("SELECT id, data, hora FROM consultas WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (paciente,))
    consultas = cursor.fetchall()

    if consultas:
        print("Consultas do paciente:")
        for i, consulta in enumerate(consultas):
            consulta_id, data, hora = consulta
            print(f"{i + 1}. Data: {data}, Hora: {hora}")

        opcao = int(input("Digite o número da consulta que deseja remarcar: "))

        if 1 <= opcao <= len(consultas):
            consulta_selecionada = consultas[opcao - 1]
            nova_data = int(input("Digite a nova data da consulta (DD/MM/AAAA): "))
            nova_hora = int(input("Digite a nova hora da consulta (HH:MM): "))

            cursor.execute("UPDATE consultas SET data=?, hora=? WHERE id=?", (nova_data, nova_hora, consulta_selecionada[0]))
            conn.commit()
            print("Consulta remarcada com sucesso.")
        else:
            print("Opção inválida.")
    else:
        print("O paciente não tem consultas registradas.")


def cancelar_consulta():
    paciente = input("Digite o nome do paciente para cancelar a consulta: ")

    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (paciente,))
    paciente_id = cursor.fetchone()

    if paciente_id:
        cursor.execute("SELECT id, medico_crm, data, hora FROM consultas WHERE paciente_id=?", (paciente_id[0],))
        consultas = cursor.fetchall()

        if consultas:
            print("Consultas do paciente:")
            for i, consulta in enumerate(consultas):
                print(f"{i + 1}. CRM: {consulta[1]}, Data: {consulta[2]}, Hora: {consulta[3]}")

            opcao = int(input("Digite o número da consulta que deseja cancelar: "))

            if 1 <= opcao <= len(consultas):
                consulta_selecionada = consultas[opcao - 1]
                consulta_id = consulta_selecionada[0]
                cursor.execute("DELETE FROM consultas WHERE id=?", (consulta_id,))
                conn.commit()
                print("Consulta cancelada com sucesso.")
            else:
                print("Opção inválida.")
        else:
            print("O paciente não tem consultas registradas.")
    else:
        print("Paciente não encontrado.")


def mostrar_consultas_paciente():
    paciente = input("Digite o nome do paciente para mostrar as consultas: ")
    cursor.execute(
        "SELECT consultas.medico_crm, consultas.data, consultas.hora FROM consultas JOIN pacientes ON consultas.paciente_id = pacientes.id WHERE pacientes.nome=?",
        (paciente,))
    consultas = cursor.fetchall()
    if consultas:
        print(f"Consultas de {paciente}:")
        for consulta in consultas:
            print(f"Médico (CRM): {consulta[0]}")
            print(f"Data da Consulta: {consulta[1]}")
            print(f"Hora da Consulta: {consulta[2]}")
    else:
        print(f"Nenhuma consulta encontrada para {paciente}.")


def cadastrar_ficha_paciente():
    paciente = input("Digite o nome do paciente para cadastrar a ficha: ")
    medico_nome = input("Digite o nome do médico: ")
    dia_hora_consulta = input("Digite o dia e hora da consulta (DDMMAAAA HHMM): ")
    doenca = input("Digite a descrição da doença: ")
    cursor.execute("SELECT id FROM pacientes WHERE nome=?", (paciente,))
    paciente_id = cursor.fetchone()
    if paciente_id:
        cursor.execute("INSERT INTO fichas (paciente_id, medico_nome, dia_hora_consulta, doenca) VALUES (?, ?, ?, ?)", (paciente_id[0], medico_nome, dia_hora_consulta, doenca))
        conn.commit()
        print("Dados cadastrado com sucesso.")
    else:
        print("Paciente não encontrado.")


def consultar_ficha_paciente():
    paciente = input("Digite o nome do paciente para consultar a ficha: ")
    cursor.execute("SELECT id, medico_nome, dia_hora_consulta, doenca FROM fichas WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (paciente,))
    fichas = cursor.fetchall()
    if fichas:
        print(f"Fichas de {paciente}:")
        for i, ficha in enumerate(fichas):
            print(f"{i + 1}. Medico: {ficha[1]}, Dia e Hora da Consulta: {ficha[2]}, Doença: {ficha[3]}")
    else:
        print(f"Nenhuma ficha encontrada para {paciente}.")

def atualizar_ficha_paciente():
    paciente = input("Digite o nome do paciente para atualizar a ficha: ")
    cursor.execute("SELECT id, medico_nome, dia_hora_consulta, doenca FROM fichas WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (paciente,))
    fichas = cursor.fetchall()
    if fichas:
        print(f"Fichas de {paciente}:")
        for i, ficha in enumerate(fichas):
            print(f"{i + 1}. Medico: {ficha[1]}, Dia e Hora da Consulta: {ficha[2]}, Doença: {ficha[3]}")

        escolha = int(input("Digite o número da ficha que deseja atualizar: "))
        if 1 <= escolha <= len(fichas):
            ficha_selecionada = fichas[escolha - 1]
            novo_nome_medico = input("Novo nome do médico: ")
            nova_data_hora_consulta = input("Nova data e hora da consulta (DD/MM/AAAA HH:MM): ")
            nova_doenca = input("Nova descrição da doença: ")

            cursor.execute("UPDATE fichas SET medico_nome=?, dia_hora_consulta=?, doenca=? WHERE id=?", (novo_nome_medico, nova_data_hora_consulta, nova_doenca, ficha_selecionada[0]))
            conn.commit()
            print("Ficha atualizada com sucesso.")
        else:
            print("Opção inválida.")
    else:
        print(f"Nenhuma ficha encontrada para {paciente}.")
def deletar_ficha_paciente():
    paciente = input("Digite o nome do paciente para deletar uma ficha: ")
    cursor.execute("SELECT id, medico_nome, dia_hora_consulta, doenca FROM fichas WHERE paciente_id IN (SELECT id FROM pacientes WHERE nome=?)", (paciente,))
    fichas = cursor.fetchall()
    if fichas:
        print(f"Fichas de {paciente}:")
        for i, ficha in enumerate(fichas):
            print(f"{i + 1}. Medico: {ficha[1]}, Dia e Hora da Consulta: {ficha[2]}, Doença: {ficha[3]}")

        escolha = int(input("Digite o número da ficha que deseja deletar: "))
        if 1 <= escolha <= len(fichas):
            ficha_selecionada = fichas[escolha - 1]
            cursor.execute("DELETE FROM fichas WHERE id=?", (ficha_selecionada[0],))
            conn.commit()
            print("Ficha deletada com sucesso.")
        else:
            print("Opção inválida.")
    else:
        print(f"Nenhuma ficha encontrada para {paciente}.")


while True:
  


    print("\nMENU:")
    print("1. Paciente")
    print("2. Plano de Saúde")
    print("3. Consulta")
    print("4. Médico")
    print("0. Sair")

    opcao = input("Escolha uma opção: ")

    if opcao == "1":
        print("\nMENU PACIENTE:")
        print("1. Cadastrar")
        print("2. Deletar")
        print("3. Consultar")
        print("4. Atualizar")
        print("5. Listar Todos os Pacientes")
        print("6. Voltar")
        sub_opcao = input("Escolha uma opção: ")
        if sub_opcao == "1":
            cadastrar_paciente()
        elif sub_opcao == "2":
            deletar_paciente()
        elif sub_opcao == "3":
            consultar_paciente()
        elif sub_opcao == "4":
            atualizar_paciente()
        elif sub_opcao == "5":
            listar_pacientes()
        elif sub_opcao == "6":
            continue

    elif opcao == "2":
        print("\nMENU PLANO DE SAÚDE:")
        print("1. Cadastrar")
        print("2. Atualizar")
        print("3. Cancelar")
        print("4. Pesquisar Plano de Saúde de Paciente")
        print("5. Listar Todos os Planos de Saúde")
        print("6. Voltar")
        sub_opcao = input("Escolha uma opção: ")
        if sub_opcao == "1":
            cadastrar_plano_de_saude()
        elif sub_opcao == "2":
            atualizar_plano_de_saude()
        elif sub_opcao == "3":
            cancelar_plano_de_saude()
        elif sub_opcao == "4":
            pesquisar_plano_de_saude()
        elif sub_opcao == "5":
            listar_planos_de_saude()
        elif sub_opcao == "6":
            continue


    elif opcao == "3":

        print("\nMENU CONSULTA:")

        print("1. Cadastrar Consulta")

        print("2. Remarcar Consulta")

        print("3. Cancelar Consulta")

        print("4. Mostrar Consultas de Paciente")
        print("5. Voltar")

        sub_opcao = input("Escolha uma opção: ")

        if sub_opcao == "1":

            cadastrar_consulta()

        elif sub_opcao == "2":

            remarcar_consulta()

        elif sub_opcao == "3":

            cancelar_consulta()

        elif sub_opcao == "4":

            mostrar_consultas_paciente()
        elif sub_opcao == "5":
            continue


    elif opcao == "4":

        print("\nMENU MÉDICO:")

        print("1.Remarcar consulta do Paciente")

        print("2. Cadastrar Ficha de Paciente")

        print("3. Consultar Ficha de Paciente")

        print("4. Atualizar Ficha de Paciente")

        print("5. Deletar Ficha de Paciente")

        print("6. Voltar")

        sub_opcao = input("Escolha uma opção: ")

        if sub_opcao == "1":

            remarcar_consulta()

        elif sub_opcao == "2":

            cadastrar_ficha_paciente()

        elif sub_opcao == "3":

            consultar_ficha_paciente()

        elif sub_opcao == "4":

            atualizar_ficha_paciente()

        elif sub_opcao == "5":

            deletar_ficha_paciente()
        elif sub_opcao == "6":
            continue


    elif opcao == "0":

        break


    else:

        print("Opção inválida. Tente novamente.")
