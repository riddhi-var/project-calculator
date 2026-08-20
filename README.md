import streamlit as st
import math

# -----------------------------
# PAGE CONFIGURATION
# -----------------------------
st.set_page_config(
    page_title="Scientific Calculator",
    page_icon="🧮",
    layout="centered"
)

# -----------------------------
# CUSTOM CSS
# -----------------------------
st.markdown("""
<style>

    .main {
        background-color: #0f172a;
    }

    .calculator-title {
        text-align: center;
        font-size: 42px;
        font-weight: bold;
        color: #38bdf8;
        margin-bottom: 5px;
    }

    .subtitle {
        text-align: center;
        color: #94a3b8;
        margin-bottom: 30px;
    }

    .display {
        background-color: #020617;
        border: 2px solid #334155;
        border-radius: 15px;
        padding: 20px;
        text-align: right;
        color: #f8fafc;
        font-size: 30px;
        margin-bottom: 20px;
    }

    div.stButton > button {
        width: 100%;
        height: 55px;
        border-radius: 12px;
        font-size: 18px;
        font-weight: bold;
        border: none;
        background-color: #1e293b;
        color: white;
        transition: 0.2s;
    }

    div.stButton > button:hover {
        background-color: #334155;
        transform: scale(1.03);
    }

    .history-title {
        color: #38bdf8;
        font-size: 24px;
        font-weight: bold;
    }

    .history-item {
        background-color: #1e293b;
        padding: 10px;
        border-radius: 8px;
        margin: 5px 0;
        color: #e2e8f0;
    }

</style>
""", unsafe_allow_html=True)


# -----------------------------
# SESSION STATE
# -----------------------------
if "expression" not in st.session_state:
    st.session_state.expression = ""

if "result" not in st.session_state:
    st.session_state.result = ""

if "history" not in st.session_state:
    st.session_state.history = []

if "angle_mode" not in st.session_state:
    st.session_state.angle_mode = "DEG"


# -----------------------------
# FUNCTIONS
# -----------------------------

def add_to_expression(value):
    st.session_state.expression += value


def clear_calculator():
    st.session_state.expression = ""
    st.session_state.result = ""


def delete_last():
    st.session_state.expression = st.session_state.expression[:-1]


def factorial(x):
    if x < 0 or int(x) != x:
        raise ValueError("Factorial is only defined for non-negative integers")
    return math.factorial(int(x))


def calculate_expression(expression):

    try:

        # Replace calculator symbols
        expression = expression.replace("×", "*")
        expression = expression.replace("÷", "/")
        expression = expression.replace("^", "**")

        # Constants
        expression = expression.replace("π", "pi")

        # -----------------------------
        # Scientific functions
        # -----------------------------

        if st.session_state.angle_mode == "DEG":

            def sin(x):
                return math.sin(math.radians(x))

            def cos(x):
                return math.cos(math.radians(x))

            def tan(x):
                return math.tan(math.radians(x))

        else:

            def sin(x):
                return math.sin(x)

            def cos(x):
                return math.cos(x)

            def tan(x):
                return math.tan(x)

        def log(x):
            return math.log10(x)

        def ln(x):
            return math.log(x)

        def sqrt(x):
            return math.sqrt(x)

        def cot(x):
            return 1 / tan(x)

        def sec(x):
            return 1 / cos(x)

        def csc(x):
            return 1 / sin(x)

        # Allowed functions/constants
        allowed = {
            "sin": sin,
            "cos": cos,
            "tan": tan,
            "cot": cot,
            "sec": sec,
            "csc": csc,
            "log": log,
            "ln": ln,
            "sqrt": sqrt,
            "factorial": factorial,
            "abs": abs,
            "pi": math.pi,
            "e": math.e
        }

        # Safe evaluation
        result = eval(
            expression,
            {"__builtins__": {}},
            allowed
        )

        # Format result
        if isinstance(result, float):

            if result.is_integer():
                return str(int(result))

            return f"{result:.12g}"

        return str(result)

    except ZeroDivisionError:
        return "Error: Division by zero"

    except ValueError:
        return "Error: Invalid value"

    except Exception:
        return "Error"


def calculate():

    if not st.session_state.expression:
        return

    result = calculate_expression(
        st.session_state.expression
    )

    st.session_state.result = result

    # Add to history
    if not result.startswith("Error"):

        calculation = (
            f"{st.session_state.expression} = {result}"
        )

        st.session_state.history.insert(
            0,
            calculation
        )

        # Keep last 10 calculations
        st.session_state.history = (
            st.session_state.history[:10]
        )


# -----------------------------
# TITLE
# -----------------------------

st.markdown(
    '<div class="calculator-title">🧮 Scientific Calculator</div>',
    unsafe_allow_html=True
)

st.markdown(
    '<div class="subtitle">Powerful calculator built with Python & Streamlit</div>',
    unsafe_allow_html=True
)


# -----------------------------
# ANGLE MODE
# -----------------------------

col1, col2 = st.columns(2)

with col1:

    mode = st.selectbox(
        "Angle Mode",
        ["DEG", "RAD"],
        index=0 if st.session_state.angle_mode == "DEG" else 1
    )

    st.session_state.angle_mode = mode


with col2:

    st.write("")
    st.write("")

    if st.button("🗑️ Clear History"):

        st.session_state.history = []

        st.rerun()


# -----------------------------
# DISPLAY
# -----------------------------

display_value = st.session_state.expression

if st.session_state.result:

    display_value = (
        st.session_state.result
    )

st.markdown(
    f"""
    <div class="display">
        {display_value if display_value else "0"}
    </div>
    """,
    unsafe_allow_html=True
)


# -----------------------------
# SCIENTIFIC BUTTONS
# -----------------------------

st.subheader("Scientific Functions")

row1 = st.columns(6)

with row1[0]:
    if st.button("sin"):
        add_to_expression("sin(")

with row1[1]:
    if st.button("cos"):
        add_to_expression("cos(")

with row1[2]:
    if st.button("tan"):
        add_to_expression("tan(")

with row1[3]:
    if st.button("log"):
        add_to_expression("log(")

with row1[4]:
    if st.button("ln"):
        add_to_expression("ln(")

with row1[5]:
    if st.button("√"):
        add_to_expression("sqrt(")


row2 = st.columns(6)

with row2[0]:
    if st.button("cot"):
        add_to_expression("cot(")

with row2[1]:
    if st.button("sec"):
        add_to_expression("sec(")

with row2[2]:
    if st.button("csc"):
        add_to_expression("csc(")

with row2[3]:
    if st.button("x!"):
        add_to_expression("factorial(")

with row2[4]:
    if st.button("π"):
        add_to_expression("π")

with row2[5]:
    if st.button("e"):
        add_to_expression("e")


# -----------------------------
# CALCULATOR BUTTONS
# -----------------------------

st.subheader("Calculator")

# Row 1
row = st.columns(4)

with row[0]:
    if st.button("AC"):
        clear_calculator()
        st.rerun()

with row[1]:
    if st.button("⌫"):
        delete_last()
        st.rerun()

with row[2]:
    if st.button("("):
        add_to_expression("(")

with row[3]:
    if st.button(")"):
        add_to_expression(")")


# Row 2
row = st.columns(4)

with row[0]:
    if st.button("7"):
        add_to_expression("7")

with row[1]:
    if st.button("8"):
        add_to_expression("8")

with row[2]:
    if st.button("9"):
        add_to_expression("9")

with row[3]:
    if st.button("÷"):
        add_to_expression("÷")


# Row 3
row = st.columns(4)

with row[0]:
    if st.button("4"):
        add_to_expression("4")

with row[1]:
    if st.button("5"):
        add_to_expression("5")

with row[2]:
    if st.button("6"):
        add_to_expression("6")

with row[3]:
    if st.button("×"):
        add_to_expression("×")


# Row 4
row = st.columns(4)

with row[0]:
    if st.button("1"):
        add_to_expression("1")

with row[1]:
    if st.button("2"):
        add_to_expression("2")

with row[2]:
    if st.button("3"):
        add_to_expression("3")

with row[3]:
    if st.button("-"):
        add_to_expression("-")


# Row 5
row = st.columns(4)

with row[0]:
    if st.button("0"):
        add_to_expression("0")

with row[1]:
    if st.button("."):
        add_to_expression(".")

with row[2]:
    if st.button("^"):
        add_to_expression("^")

with row[3]:
    if st.button("+"):
        add_to_expression("+")


# -----------------------------
# EQUAL BUTTON
# -----------------------------

st.write("")

if st.button("=", type="primary", use_container_width=True):

    calculate()

    st.rerun()


# -----------------------------
# RESULT
# -----------------------------

if st.session_state.result:

    st.success(
        f"Result: {st.session_state.result}"
    )


# -----------------------------
# HISTORY
# -----------------------------

st.markdown(
    '<div class="history-title">📜 Calculation History</div>',
    unsafe_allow_html=True
)

if st.session_state.history:

    for item in st.session_state.history:

        st.markdown(
            f"""
            <div class="history-item">
                {item}
            </div>
            """,
            unsafe_allow_html=True
        )

else:

    st.info("No calculations yet.")


# -----------------------------
# FOOTER
# -----------------------------

st.write("")

st.markdown(
    """
    <div style="text-align:center;color:#64748b;">
        Built with ❤️ using Python & Streamlit
    </div>
    """,
    unsafe_allow_html=True
)
