import tkinter as tk, csv, os

ARQ = "agendamentos.csv"
if not os.path.exists(ARQ):
    with open(ARQ, "w", newline="", encoding="utf-8") as f:
        f.write("nome,data,hora,servico\n")

GRADE = ["09:00","09:30","10:00","10:30","11:00","11:30","12:00","12:30"]
etapa = 0
nome = data = hora = ""

def salvar(n, d, h, s):
    with open(ARQ, "a", newline="", encoding="utf-8") as f:
        csv.writer(f).writerow([n, d, h, s])

def carregar():
    with open(ARQ, "r", encoding="utf-8") as f:
        return list(csv.DictReader(f))

def processar(msg):
    global etapa, nome, data, hora
    m = msg.lower()

    if m == "marcar":
        etapa = 1
        return "Nome do cliente?"

    if etapa == 1:
        nome = msg
        etapa = 2
        return "Data (AAAA-MM-DD)?"

    if etapa == 2:
        data = msg
        etapa = 3
        return f"Hora ({', '.join(GRADE)})?"

    if etapa == 3:
        if msg not in GRADE:
            return "Hora inválida!"
        hora = msg
        etapa = 4
        return "Serviço?"

    if etapa == 4:
        salvar(nome, data, hora, m)
        etapa = 0
        return f"Agendado: {nome} - {data} {hora} - {m}"

    if m == "listar":
        ag = carregar()
        if not ag:
            return "Nenhum agendamento."
        return "\n".join(f"{a['data']} {a['hora']} - {a['nome']} ({a['servico']})" for a in ag[-5:])

    return "Use: marcar ou listar"

def enviar():
    t = entrada.get()
    chat.insert(tk.END, f"Você: {t}\n")
    entrada.delete(0, tk.END)
    r = processar(t)
    chat.insert(tk.END, f"Bot: {r}\n\n")

jan = tk.Tk()
jan.title("Chatbot Espaço Leia")
chat = tk.Text(jan, width=60, height=20)
chat.pack()
entrada = tk.Entry(jan, width=50)
entrada.pack()
tk.Button(jan, text="Enviar", command=enviar).pack()
jan.mainloop()
