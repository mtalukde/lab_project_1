# lab_project_1
1. Purpose of the work. The purpose of the laboratory work is to develop the following skills: -Work with user-defined functions; -Ability to use cycles; -Ability to work with the conditional operator; -Exploring formatted data output; -Exploring the math library math_h.
#include <iostream>
#include <cmath>
#include <iomanip>
#include <string>
#include <vector>
#include<conio.h>

using namespace std;

// Класс для хранения результата вычислений
struct CalculationResult {
    double x;
    double y;
    string status;
    bool hasError;
};

// Аналитическая проверка: может ли быть вычислено значение?
pair<bool, string> canCalculate(double x, double a) {
    double t = a + x / 3.0;
    double tan_t = tan(t);

    // Проверка на переполнение tan
    if (isinf(tan_t)) {
        return {false, "tan() overflow"};
    }

    // Проверка аргумента корня
    double arg_sqrt = a + tan_t;
    if (arg_sqrt < 0) {
        return {false, "sqrt of negative"};
    }

    // Проверка деления на ноль
    double sqrt_val = sqrt(arg_sqrt);
    if (fabs(sqrt_val - 1.0) < 1e-12) {
        return {false, "division by zero"};
    }

    return {true, ""};
}

// Вычисление значения функции
CalculationResult calculateForX(double x, double a) {
    CalculationResult result;
    result.x = x;

    auto [canCalc, errorMsg] = canCalculate(x, a);

    if (!canCalc) {
        result.hasError = true;
        result.status = errorMsg;
        result.y = NAN;
        return result;
    }

    // Все проверки пройдены, вычисляем значение
    double t = a + x / 3.0;
    double tan_t = tan(t);
    double arg_sqrt = a + tan_t;
    double sqrt_val = sqrt(arg_sqrt);
    result.y = 1.0 / (1.0 - sqrt_val);
    result.hasError = false;
    result.status = "OK";

    return result;
}

// Поиск критических x для деления на ноль (для информации)
void printCriticalPoints(double a) {
    cout << "\n=== Аналитический анализ для a = " << a << " ===" << endl;

    // Уравнение для деления на ноль: tan(a + x/3) = 1 - a
    double right_side = 1 - a;

    cout << "Уравнение для деления на ноль: tan(a + x/3) = " << right_side << endl;

    if (fabs(right_side) <= 1.0) {
        // Основное решение
        double principal = atan(right_side);
        cout << "Основные критические точки x:" << endl;

        for (int n = -2; n <= 2; n++) {
            double x_crit = 3 * (principal - a + M_PI * n);
            if (x_crit >= -1.7 && x_crit <= 3.3) {
                cout << "  x ≈ " << fixed << setprecision(3) << x_crit;

                // Проверим, действительно ли здесь деление на ноль
                double t = a + x_crit / 3.0;
                double tan_t = tan(t);
                double arg_sqrt = a + tan_t;
                if (fabs(arg_sqrt - 1.0) < 1e-6) {
                    cout << "  [ВНУТРИ ИНТЕРВАЛА!]" << endl;
                } else {
                    cout << endl;
                }
            }
        }
    } else {
        cout << "Уравнение tan(a + x/3) = " << right_side << " может не иметь решений" << endl;
    }

    cout << "=====================================\n" << endl;
}

int main() {
    // Параметры варианта 24
    const double x_start = -1.7;
    const double x_end = 3.3;
    const double dx = 0.5;

    double a;

    cout << "========================================================" << endl;
    cout << "ЛАБОРАТОРНАЯ РАБОТА 1. ВАРИАНТ 24" << endl;
    cout << "y = 1 / [1 - sqrt(a + tan(a + x/3))]" << endl;
    cout << "Интервал: x ∈ [" << x_start << ", " << x_end << "], шаг = " << dx << endl;
    cout << "========================================================" << endl;

    cout << "Введите параметр a: ";
    cin >> a;


    printCriticalPoints(a);

    vector<CalculationResult> results;

    // Вычисление для всех x
    int pointCount = 0;
    for (double x = x_start; x <= x_end + 1e-9; x += dx) {
        pointCount++;
        results.push_back(calculateForX(x, a));
    }

    size_t maxStatusLen = 0;
    for (const auto& res : results) {
        if (res.status.length() > maxStatusLen) {
            maxStatusLen = res.status.length();
        }
    }
    maxStatusLen = max(maxStatusLen, (size_t)15);


    const int col1 = 8;    // № п/п
    const int col2 = 12;   // x
    const int col3 = 12;   // a
    const int col4 = maxStatusLen + 5;  // y / статус

    // Шапка таблицы
    cout << "\n" << string(col1 + col2 + col3 + col4 + 9, '=') << endl;
    cout << left
         << setw(col1) << "№ п/п"
         << "│ " << setw(col2) << "x"
         << "│ " << setw(col3) << "a"
         << "│ " << setw(col4) << "y / статус" << endl;
    cout << string(col1 + col2 + col3 + col4 + 9, '-') << endl;

    // Данные таблицы
    for (size_t i = 0; i < results.size(); i++) {
        const auto& res = results[i];

        cout << left << setw(col1) << (i + 1);
        cout << "│ " << fixed << setprecision(3) << setw(col2) << res.x;
        cout << "│ " << setw(col3) << a;

        if (res.hasError) {
            // Вывод сообщения об ошибке
            cout << "│ " << left << setw(col4) << res.status;
        } else {
            // Вывод вычисленного значения
            if (isinf(res.y)) {
                cout << "│ " << left << setw(col4) << "infinity";
            } else if (isnan(res.y)) {
                cout << "│ " << left << setw(col4) << "calculation error";
            } else {
                cout << "│ " << fixed << setprecision(6) << setw(col4) << res.y;
            }
        }
        cout << endl;
    }

    cout << string(col1 + col2 + col3 + col4 + 9, '=') << endl;

    // Статистика
    int errorCount = 0;
    for (const auto& res : results) {
        if (res.hasError) errorCount++;
    }

    cout << "\nСТАТИСТИКА:" << endl;
    cout << "Всего точек: " << results.size() << endl;
    cout << "Успешно вычислено: " << (results.size() - errorCount) << endl;
    cout << "Ошибок вычисления: " << errorCount << endl;

    if (errorCount == results.size()) {
        cout << "ВНИМАНИЕ: Невозможно вычислить ни одного значения!" << endl;
        cout << "Таблица всё равно выведена полностью - это соответствует требованию." << endl;
    }

    getch();
    return 0;
}

