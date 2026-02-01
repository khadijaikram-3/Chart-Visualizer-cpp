#include <graphics.h>
#include <windows.h>
#include <iostream>
#include <cstring>
#include <cstdlib>
#include <shlobj.h>    // For SHGetFolderPathA
#include <fstream>
#include <string>
#include <sstream>  // For std::ostringstream
#include <vector>
using namespace std;

// Global save folder path variable accessible across your code
std::string saveFolderPath;

// Global chart data
int n;
char labels[10][20];
int values[10];

// Additional array for stacked bar values [bar][stack_idx]
int stackedValues[10][3];

const int colors[] = {RED, GREEN, BLUE, CYAN, MAGENTA, YELLOW, LIGHTBLUE, LIGHTGREEN, LIGHTRED, BROWN};

int currentChartType = 0;  // Track currently selected chart type
// Forward declarations
void getChartData(int centerX);
int showChartMenu(int centerX);
void getStackedBarData(int centerX);
// Base class for all charts
class Chart {
public:
    virtual int draw(int centerX) = 0;
    virtual ~Chart() {}
protected:
    void drawLegend(int x, int y) {
        int squareSize = 15;
        settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
        outtextxy(x, y - 30, (char*)"Legend:");
        for (int i = 0; i < n; ++i) {
            setfillstyle(SOLID_FILL, colors[i % 10]);
            bar(x, y + i * 30, x + squareSize, y + i * 30 + squareSize);
            char buf[100];
            if (currentChartType == 9) {  // Stacked Bar Chart
                sprintf(buf, "%s (S1:%d, S2:%d, S3:%d)", labels[i], stackedValues[i][0], stackedValues[i][1], stackedValues[i][2]);
            }
            else {
                sprintf(buf, "%s (%d)", labels[i], values[i]);
            }
            outtextxy(x + squareSize + 10, y + i * 30, buf);
        }
    }
};


// Navigation buttons, now 4 buttons including "Save Data"
int drawNavigationButtons() {
    int btnWidth = 280, btnHeight = 40, spacing = 15;
    int startX = getmaxx() - btnWidth - 30;       // Right align
    int startY = getmaxy() - 4 * (btnHeight + spacing) - 30;  // Adjust for 4 buttons

    const char* buttons[] = {
        "Back to Chart Selection",
        "Re-enter Data",
        "Return to Welcome",
        "Save Data"
    };

    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 1);
    for (int i = 0; i < 4; ++i) {
        setfillstyle(SOLID_FILL, LIGHTCYAN);
        bar(startX, startY + i * (btnHeight + spacing), startX + btnWidth, startY + i * (btnHeight + spacing) + btnHeight);
        setcolor(BLACK);
        rectangle(startX, startY + i * (btnHeight + spacing), startX + btnWidth, startY + i * (btnHeight + spacing) + btnHeight);
        outtextxy(startX + 10, startY + i * (btnHeight + spacing) + 10, (char*)buttons[i]);
    }

    int mx, my;
    while (true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);
            for (int i = 0; i < 4; ++i) {
                int top = startY + i * (btnHeight + spacing);
                int bottom = top + btnHeight;
                if (mx >= startX && mx <= startX + btnWidth && my >= top && my <= bottom)
                    return i + 1;  // Return button index 1..4
            }
        }
        delay(50);
    }
}

// (Chart classes like BarChart, PieChart, LineChart, DonutChart, ScatterPlot, AreaChart, ColumnChart, HistogramChart, StackedBarChart, MultiLineChart)

// Bar Chart Class
class BarChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 1;
        cleardevice(); setbkcolor(WHITE); cleardevice();
        int xAxisY = getmaxy() - 150, yAxisX = 100, chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(yAxisX, 100, yAxisX, xAxisY);
        line(yAxisX, xAxisY, getmaxx() - 100, xAxisY);

        int maxVal = 0;
        for (int i = 0; i < n; i++) if (values[i] > maxVal) maxVal = values[i];
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = xAxisY - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(yAxisX - 40, y - 5, valStr);
            line(yAxisX - 5, y, yAxisX, y);
        }

        int barWidth = 40, gap = 30, startX = yAxisX + 50;
        for (int i = 0; i < n; ++i) {
            int bh = (values[i] * chartHeight) / yMax;
            setfillstyle(SOLID_FILL, colors[i % 10]);
            bar(startX, xAxisY - bh, startX + barWidth, xAxisY);
            startX += barWidth + gap;
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 100, 40,(char*) "Bar Chart");
        return drawNavigationButtons();
    }
};

// Pie Chart Class
class PieChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 2;
        cleardevice(); setbkcolor(WHITE); cleardevice();
        int total = 0;
        for (int i = 0; i < n; i++) total += values[i];
        if (total == 0) total = 1;

        int radius = 200, cx = getmaxx() / 2, cy = getmaxy() / 2, angle = 0;
        for (int i = 0; i < n; ++i) {
            float perc = (float)values[i] / total;
            int sweep = (int)(perc * 360 + 0.5);
            setfillstyle(SOLID_FILL, colors[i % 10]);
            pieslice(cx, cy, angle, angle + sweep, radius);
            angle += sweep;
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 80, 50, (char*)"Pie Chart");
        return drawNavigationButtons();
    }
};

// Line Chart Class
class LineChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 3;
        cleardevice(); setbkcolor(WHITE); cleardevice();
        int xAxisY = getmaxy() - 150, yAxisX = 100, chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(yAxisX, 100, yAxisX, xAxisY);
        line(yAxisX, xAxisY, getmaxx() - 100, xAxisY);

        int maxVal = 0;
        for (int i = 0; i < n; i++) if (values[i] > maxVal) maxVal = values[i];
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = xAxisY - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(yAxisX - 40, y - 5, valStr);
            line(yAxisX - 5, y, yAxisX, y);
        }

        int gap = 80, startX = yAxisX + 50;
        int prevX = -1, prevY = -1;

        for (int i = 0; i < n; ++i) {
            int x = startX + i * gap;
            int y = xAxisY - (values[i] * chartHeight / yMax);

            setcolor(colors[i % 10]);
            setfillstyle(SOLID_FILL, colors[i % 10]);
            fillellipse(x, y, 5, 5);

            if (i > 0)
                line(prevX, prevY, x, y);

            prevX = x;
            prevY = y;
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 100, 40, (char*)"Line Chart");
        return drawNavigationButtons();
    }
};

// Donut Chart Class
class DonutChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 4;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int total = 0;
        for (int i = 0; i < n; i++) total += values[i];
        if (total == 0) total = 1;

        int radiusOuter = 200;
        int radiusInner = 100;
        int cx = getmaxx() / 2, cy = getmaxy() / 2;
        int angle = 0;

        for (int i = 0; i < n; ++i) {
            float perc = (float)values[i] / total;
            int sweep = (int)(perc * 360 + 0.5);
            setfillstyle(SOLID_FILL, colors[i % 10]);
            pieslice(cx, cy, angle, angle + sweep, radiusOuter);
            angle += sweep;
        }

        setfillstyle(SOLID_FILL, WHITE);
        fillellipse(cx, cy, radiusInner, radiusInner);
        setcolor(BLACK);
        circle(cx, cy, radiusInner);

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 100, 50, (char*)"Donut Chart");

        return drawNavigationButtons();
    }
};

// Scatter Plot Class
class ScatterPlot : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 5;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xOrigin = 100, yOrigin = getmaxy() - 150;
        int chartWidth = getmaxx() - 200;
        int chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(xOrigin, 100, xOrigin, yOrigin);
        line(xOrigin, yOrigin, xOrigin + chartWidth, yOrigin);

        int maxVal = 0;
        for (int i = 0; i < n; ++i) {
            if (values[i] > maxVal) maxVal = values[i];
        }
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = yOrigin - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(xOrigin - 40, y - 5, valStr);
            line(xOrigin - 5, y, xOrigin, y);
        }

        int gapX = chartWidth / (n + 1);
        for (int i = 0; i < n; ++i) {
            int x = xOrigin + (i + 1) * gapX;
            int y = yOrigin - (values[i] * chartHeight / yMax);
            setcolor(colors[i % 10]);
            setfillstyle(SOLID_FILL, colors[i % 10]);
            fillellipse(x, y, 5, 5);

            setcolor(BLACK);
            outtextxy(x - 10, y - 20, labels[i]);
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 120, 40, (char*)"Scatter Plot");

        return drawNavigationButtons();
    }
};

// Area Chart Class
class AreaChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 6;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xOrigin = 100, yOrigin = getmaxy() - 150;
        int chartWidth = getmaxx() - 200;
        int chartHeight = 300;

        setcolor(BLACK);
        setlinestyle(SOLID_LINE, 0, 2);
        line(xOrigin, 100, xOrigin, yOrigin);
        line(xOrigin, yOrigin, xOrigin + chartWidth, yOrigin);

        int maxVal = 0;
        for (int i = 0; i < n; i++)
            if (values[i] > maxVal) maxVal = values[i];
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = yOrigin - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(xOrigin - 40, y - 5, valStr);
            line(xOrigin - 5, y, xOrigin, y);
        }

        int gapX = chartWidth / (n + 1);

        int firstX = xOrigin + gapX;
        int firstY = yOrigin - (values[0] * chartHeight / yMax);
        int firstPoly[] = {
            xOrigin, yOrigin,
            firstX, yOrigin,
            firstX, firstY,
            xOrigin, yOrigin
        };
        setfillstyle(SOLID_FILL, colors[0 % 10]);
        fillpoly(4, firstPoly);

        for (int i = 1; i < n; ++i) {
            int prevX = xOrigin + i * gapX;
            int prevY = yOrigin - (values[i - 1] * chartHeight / yMax);
            int currX = xOrigin + (i + 1) * gapX;
            int currY = yOrigin - (values[i] * chartHeight / yMax);

            int poly[] = {
                prevX, yOrigin,
                prevX, prevY,
                currX, currY,
                currX, yOrigin
            };
            setfillstyle(SOLID_FILL, colors[i % 10]);
            fillpoly(4, poly);
        }

        for (int i = 0; i < n; ++i) {
            int x = xOrigin + (i + 1) * gapX;
            int y = yOrigin - (values[i] * chartHeight / yMax);
            setcolor(BLACK);
            outtextxy(x - 10, y - 20, labels[i]);
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY); outtextxy(centerX - 100, 40, (char*)"Area Chart");

        return drawNavigationButtons();
    }
};

// Column Chart Class
class ColumnChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 7;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xOrigin = 100, yOrigin = getmaxy() - 150;
        int chartWidth = getmaxx() - 200;
        int chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(xOrigin, 100, xOrigin, yOrigin);
        line(xOrigin, yOrigin, xOrigin + chartWidth, yOrigin);

        int maxVal = 0;
        for (int i = 0; i < n; ++i)
            if (values[i] > maxVal) maxVal = values[i];
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = yOrigin - (val * chartHeight / yMax);
            char buf[10]; itoa(val, buf, 10);
            outtextxy(xOrigin - 40, y - 5, buf);
            line(xOrigin - 5, y, xOrigin, y);
        }

        int columnWidth = 25;
        int gap = (chartWidth - (n * columnWidth)) / (n + 1);
        int x = xOrigin + gap;

        for (int i = 0; i < n; ++i) {
            int h = (values[i] * chartHeight) / yMax;
            int topY = yOrigin - h;

            setfillstyle(SOLID_FILL, colors[i % 10]);
            bar(x, topY, x + columnWidth, yOrigin);

            setcolor(BLACK);
            outtextxy(x, topY - 20, labels[i]);

            x += columnWidth + gap;
        }

        drawLegend(getmaxx() - 300, 100);

        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY);
        outtextxy(centerX - 120, 40, (char*)"Column Chart");

        return drawNavigationButtons();
    }
};

// Histogram Chart Class
class HistogramChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 8;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xOrigin = 100, yOrigin = getmaxy() - 150;
        int chartWidth = getmaxx() - 200;
        int chartHeight = 300;

        int maxVal = 0;
        for (int i = 0; i < n; ++i)
            if (values[i] > maxVal) maxVal = values[i];
        if (maxVal == 0) maxVal = 1;

        int binSize = 10;
        int binCount = (maxVal + binSize - 1) / binSize;
        int binFreq[20] = {0};

        for (int i = 0; i < n; ++i) {
            int binIdx = values[i] / binSize;
            if (binIdx >= 20) binIdx = 19;
            binFreq[binIdx]++;
        }

        int maxFreq = 0;
        for (int i = 0; i < binCount; ++i)
            if (binFreq[i] > maxFreq) maxFreq = binFreq[i];
        if (maxFreq == 0) maxFreq = 1;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(xOrigin, 100, xOrigin, yOrigin);
        line(xOrigin, yOrigin, xOrigin + chartWidth, yOrigin);

        int yMax = ((maxFreq + 4) / 5) * 5;
        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = yOrigin - (val * chartHeight / yMax);
            char buf[10]; itoa(val, buf, 10);
            outtextxy(xOrigin - 40, y - 5, buf);
            line(xOrigin - 5, y, xOrigin, y);
        }

        int barWidth = 40, gap = 20;
        int x = xOrigin + gap;

        for (int i = 0; i < binCount; ++i) {
            int barHeight = (binFreq[i] * chartHeight) / yMax;
            int topY = yOrigin - barHeight;
            setfillstyle(SOLID_FILL, colors[i % 10]);
            bar(x, topY, x + barWidth, yOrigin);

            char label[20];
            sprintf(label, "%d-%d", i * binSize, (i + 1) * binSize - 1);
            setcolor(BLACK);
            outtextxy(x, yOrigin + 10, label);

            x += barWidth + gap;
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY);
        outtextxy(centerX - 120, 40,(char*) "Histogram Chart");

        return drawNavigationButtons();
    }
};

// Stacked Bar Chart Class
class StackedBarChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 9;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xAxisY = getmaxy() - 150, yAxisX = 100;
        int chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(yAxisX, 100, yAxisX, xAxisY);
        line(yAxisX, xAxisY, getmaxx() - 100, xAxisY);

        int maxVal = 0;
        for (int i = 0; i < n; i++) {
            int totalStack = stackedValues[i][0] + stackedValues[i][1] + stackedValues[i][2];
            if (totalStack > maxVal) maxVal = totalStack;
        }
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = xAxisY - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(yAxisX - 40, y - 5, valStr);
            line(yAxisX - 5, y, yAxisX, y);
        }

        int barWidth = 40, gap = 30, startX = yAxisX + 50;
        for (int i = 0; i < n; ++i) {
            int stack1Height = stackedValues[i][0] * chartHeight / yMax;
            int stack2Height = stackedValues[i][1] * chartHeight / yMax;
            int stack3Height = stackedValues[i][2] * chartHeight / yMax;
            int yTop = xAxisY - (stack1Height + stack2Height + stack3Height);

            setfillstyle(SOLID_FILL, LIGHTBLUE);
            bar(startX, xAxisY - stack1Height, startX + barWidth, xAxisY);

            setfillstyle(SOLID_FILL, LIGHTGREEN);
            bar(startX, xAxisY - stack1Height - stack2Height, startX + barWidth, xAxisY - stack1Height);

            setfillstyle(SOLID_FILL, LIGHTRED);
            bar(startX, yTop, startX + barWidth, xAxisY - stack1Height - stack2Height);

            setcolor(BLACK);
            outtextxy(startX, xAxisY + 10, labels[i]);

            startX += barWidth + gap;
        }

        drawLegend(getmaxx() - 300, 100);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY);
        outtextxy(centerX - 140, 40,(char*) "Stacked Bar Chart");

        return drawNavigationButtons();
    }
};

// Multi-Line Chart Class
class MultiLineChart : public Chart {
public:
    int draw(int centerX) override {
        currentChartType = 10;
        cleardevice(); setbkcolor(WHITE); cleardevice();

        int xAxisY = getmaxy() - 150, yAxisX = 100;
        int chartHeight = 300;

        setcolor(BLACK); setlinestyle(SOLID_LINE, 0, 2);
        line(yAxisX, 100, yAxisX, xAxisY);
        line(yAxisX, xAxisY, getmaxx() - 100, xAxisY);

        int maxVal = 0;
        for (int i = 0; i < n; i++)
            if (values[i] > maxVal) maxVal = values[i];
        int yMax = ((maxVal + 4) / 5) * 5;
        if (yMax == 0) yMax = 5;

        for (int i = 0; i <= 5; ++i) {
            int val = i * yMax / 5;
            int y = xAxisY - (val * chartHeight / yMax);
            char valStr[10]; itoa(val, valStr, 10);
            outtextxy(yAxisX - 40, y - 5, valStr);
            line(yAxisX - 5, y, yAxisX, y);
        }

        int lineColors[] = {RED, GREEN, BLUE};
        const char* lineNames[] = {"Line A", "Line B", "Line C"};

        int gap = 70;
        int startX = yAxisX + 50;

        for (int lineIndex = 0; lineIndex < 3; ++lineIndex) {
            setcolor(lineColors[lineIndex]);
            setlinestyle(SOLID_LINE, 0, 2);

            int prevX = -1, prevY = -1;

            for (int i = 0; i < n; ++i) {
                int x = startX + i * gap;
                int val = values[i];

                int subVal = (val * (60 + 20 * lineIndex)) / 100;
                int y = xAxisY - (subVal * chartHeight / yMax);

                if (prevX != -1)
                    line(prevX, prevY, x, y);

                circle(x, y, 3);
                prevX = x;
                prevY = y;
            }
        }

        setcolor(BLACK);
        for (int i = 0; i < n; ++i) {
            int x = startX + i * gap;
            outtextxy(x - 10, xAxisY + 10, labels[i]);
        }

        settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
        int lx = getmaxx() - 300, ly = 100;
        outtextxy(lx, ly - 30,(char*) "Legend:");
        for (int i = 0; i < 3; ++i) {
            setcolor(lineColors[i]);
            line(lx, ly + i * 30, lx + 20, ly + i * 30);
            setcolor(BLACK);
            outtextxy(lx + 30, ly + i * 30 - 5, (char*)lineNames[i]);
        }

        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
        setcolor(DARKGRAY);
        outtextxy(centerX - 130, 40,(char*) "Multi-Line Chart");

        return drawNavigationButtons();
    }
};

// Function declarations for load/save and input functions
int showSaveScreen(int centerX);
bool handleNav(int nav, int centerX);
bool showLoadOrNewDataScreen(int centerX);
bool loadChartFromFile(const std::string& filepath);

// Function to get Desktop path and create ChartVisualizerData folder on Desktop
std::string getSaveFolderPath() {
    char desktopPath[MAX_PATH];
    HRESULT hr = SHGetFolderPathA(NULL, CSIDL_DESKTOP, NULL, SHGFP_TYPE_CURRENT, desktopPath);
    if (FAILED(hr)) {
        // Fallback path if failed
        return ".\\ChartVisualizerData\\";
    }
    std::string folder(desktopPath);
    folder += "\\ChartVisualizerData\\";
    // Create directory if not exists
    if (CreateDirectoryA(folder.c_str(), NULL) || GetLastError() == ERROR_ALREADY_EXISTS) {
        return folder;
    } else {
        // Could not create, fallback
        return ".\\ChartVisualizerData\\";
    }
}

std::vector<std::string> listSavedFiles(const std::string& directory) {
    std::vector<std::string> files;

    std::string searchPath = directory + "\\*.txt";
    WIN32_FIND_DATAA findData;
    HANDLE hFind = FindFirstFileA(searchPath.c_str(), &findData);

    if (hFind == INVALID_HANDLE_VALUE) {
        return files;  // empty
    }

    do {
        if (!(findData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)) {
            files.push_back(findData.cFileName);
        }
    } while (FindNextFileA(hFind, &findData) != 0);

    FindClose(hFind);
    return files;
}


// Saves current chart data to a file with given filename inside saveFolderPath
// stackedValues parameter used only for Stacked Bar chart (chartType == 9)
bool saveChartToFile(const std::string& filename, int chartType,
                    int n, char labels[][20], int values[], int stackedValues[][3]) {
    if (filename.empty()) return false;

    std::string fullPath = saveFolderPath + filename;

    std::ofstream outFile(fullPath);
    if (!outFile.is_open()) return false;

    outFile << "ChartType:" << chartType << "\n";
    outFile << "Count:" << n << "\n";

    // Write Labels comma separated
    for (int i = 0; i < n; ++i) {
        outFile << labels[i];
        if (i < n - 1) outFile << ",";
    }
    outFile << "\n";

    // Write values (for stacked bar sum stacks, else just values)
    for (int i = 0; i < n; ++i) {
        if (chartType == 9) {
            int sumStacks = stackedValues[i][0] + stackedValues[i][1] + stackedValues[i][2];
            outFile << sumStacks;
        } else {
            outFile << values[i];
        }
        if (i < n - 1) outFile << ",";
    }
    outFile << "\n";

    // If stacked bar, write stacks line by line
    if (chartType == 9) {
        for (int i = 0; i < n; ++i) {
            outFile << stackedValues[i][0] << "," << stackedValues[i][1] << "," << stackedValues[i][2] << "\n";
        }
    }

    outFile.close();
    return true;
}


//------------- showSaveScreen implementation with save to file and screenshot -----------
int showSaveScreen(int centerX) {
    using namespace std;
    cleardevice();
    setbkcolor(WHITE);
    cleardevice();

    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
    setcolor(BLACK);
    outtextxy(centerX - 150, 30, (char*)"Save Data Screen");

    // Draw buttons for: 1. Save existing, 2. Save new, 3. Exit
    const char* options[] = { "Save to Existing File", "Save to New File", "Exit Save Screen" };
    int btnWidth = 500, btnHeight = 50, spacing = 30;
    int startY = 100, startX = centerX - btnWidth / 2;

    for (int i = 0; i < 3; i++) {
        setfillstyle(SOLID_FILL, MAGENTA);
        bar(startX, startY + i * (btnHeight + spacing), startX + btnWidth, startY + i * (btnHeight + spacing) + btnHeight);
        setcolor(BLACK);
        rectangle(startX, startY + i * (btnHeight + spacing), startX + btnWidth, startY + i * (btnHeight + spacing) + btnHeight);
        outtextxy(startX + 10, startY + i * (btnHeight + spacing) + 15, (char*)options[i]);
    }

    int mx, my;
    while (true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);

            for (int i = 0; i < 3; i++) {
                int top = startY + i * (btnHeight + spacing);
                int bottom = top + btnHeight;
                if (mx >= startX && mx <= startX + btnWidth && my >= top && my <= bottom) {
                    clearmouseclick(WM_LBUTTONDOWN);

                    if (i == 0) {  // Save existing file
                        vector<string> files = listSavedFiles(saveFolderPath);
                        if (files.empty()) {
                            cleardevice();
                            setcolor(RED);
                            settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
                            outtextxy(centerX - 200, getmaxy() / 2, (char*)"No saved files found.");
                            delay(1500);
                            return 0;
                        }

                        int listStartY = 100;
                        int fileBtnWidth = 450, fileBtnHeight = 40, fileSpacing = 10;
                        cleardevice();
                        setbkcolor(WHITE);
                        setcolor(BLACK);
                        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
                        outtextxy(centerX - 180, 30, (char*)"Select file to overwrite");

                        while (true) {
                            for (int j = 0; j < (int)files.size(); j++) {
                                int ypos = listStartY + j * (fileBtnHeight + fileSpacing);
                                setfillstyle(SOLID_FILL, LIGHTCYAN);
                                bar(centerX - fileBtnWidth / 2, ypos, centerX + fileBtnWidth / 2, ypos + fileBtnHeight);
                                setcolor(BLACK);
                                rectangle(centerX - fileBtnWidth / 2, ypos, centerX + fileBtnWidth / 2, ypos + fileBtnHeight);
                                outtextxy(centerX - fileBtnWidth / 2 + 10, ypos + 10, (char*)files[j].c_str());
                            }

                            if (ismouseclick(WM_LBUTTONDOWN)) {
                                getmouseclick(WM_LBUTTONDOWN, mx, my);
                                for (int j = 0; j < (int)files.size(); j++) {
                                    int ypos = listStartY + j * (fileBtnHeight + fileSpacing);
                                    if (mx >= centerX - fileBtnWidth / 2 && mx <= centerX + fileBtnWidth / 2 &&
                                        my >= ypos && my <= ypos + fileBtnHeight) {
                                        clearmouseclick(WM_LBUTTONDOWN);

                                        cleardevice();
                                        setcolor(BLACK);
                                        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
                                        string confirm = "Overwrite file: " + files[j] + "? Click again to confirm.";
                                        outtextxy(centerX - 250, getmaxy() / 2, (char*)confirm.c_str());

                                        while (!ismouseclick(WM_LBUTTONDOWN)) delay(50);
                                        clearmouseclick(WM_LBUTTONDOWN);

                                        bool success = saveChartToFile(files[j], currentChartType, n, labels, values, stackedValues);

                                        cleardevice();
                                        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
                                        if (success)
                                            outtextxy(centerX - 150, getmaxy() / 2, (char*)"File saved successfully.");
                                        else
                                            outtextxy(centerX - 150, getmaxy() / 2, (char*)"Failed to save file.");
                                        delay(1500);
                                        return 0;
                                    }
                                }
                            }
                            delay(50);
                        }
                    }
                    else if (i == 1) {  // Save new file
                        cleardevice();
                        setbkcolor(WHITE);
                        setcolor(BLACK);
                        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
                        outtextxy(centerX - 150, 30, (char*)"Enter new filename (.txt):");
                        const int boxX = centerX - 150;
                        const int boxY = getmaxy() / 2;
                        const int boxW = 300;
                        const int boxH = 40;
                        char filename[100] = {};
                        int len = 0;

                        rectangle(boxX, boxY, boxX + boxW, boxY + boxH);

                        while (true) {
                            setfillstyle(SOLID_FILL, WHITE);
                            bar(boxX + 1, boxY + 1, boxX + boxW - 1, boxY + boxH - 1);
                            outtextxy(boxX + 5, boxY + 10, filename);

                            if (kbhit()) {
                                char ch = getch();
                                if (ch == 13) {  // Enter
                                    if (len == 0) continue;
                                    string fn(filename);
                                    if (fn.size() < 4 || fn.substr(fn.size() - 4) != ".txt") {
                                        fn += ".txt";
                                    }
                                    bool success = saveChartToFile(fn, currentChartType, n, labels, values, stackedValues);

                                    cleardevice();
                                    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
                                    if (success)
                                        outtextxy(centerX - 150, getmaxy() / 2, (char*)"File saved successfully.");
                                    else
                                        outtextxy(centerX - 150, getmaxy() / 2, (char*)"Failed to save file.");
                                    delay(1500);
                                    return 0;
                                }
                                else if (ch == 8 && len > 0) {
                                    filename[--len] = 0;
                                }
                                else if (len < 95 && ((ch >= 32 && ch <= 126))) {
                                    filename[len++] = ch;
                                    filename[len] = '\0';
                                }
                            }
                            delay(50);
                        }
                    }
                    else if (i == 2) {
                        return 1; // Exit save screen
                    }
                }
            }
        }
        delay(50);
    }
    return 0;
}

// Implementation of handleNav function
bool handleNav(int nav, int centerX) {
    if (nav == 1) return false;
    else if (nav == 2) {
        if (currentChartType == 9)
            getStackedBarData(centerX);
        else
            getChartData(centerX);
        return false;
    }
    else if (nav == 3) return true;
    else if (nav == 4) {
        return showSaveScreen(centerX) == 0 ? false : true;
    }
    return false;
}

// Implementation of loadChartFromFile
bool loadChartFromFile(const std::string& filepath) {
    std::ifstream inFile(filepath);
    if (!inFile.is_open()) return false;

    std::string line;

    // Read chartType line
    if (!std::getline(inFile, line)) return false;
    if (line.find("ChartType:") != 0) return false;
    currentChartType = std::stoi(line.substr(10));

    // Read count N
    if (!std::getline(inFile, line)) return false;
    if (line.find("Count:") != 0) return false;
    n = std::stoi(line.substr(6));
    if (n <= 0 || n > 10) return false;

    // Read labels line (comma separated)
    if (!std::getline(inFile, line)) return false;
    std::istringstream ssLabels(line);
    std::string lbl;
    int lblIndex = 0;
    while (std::getline(ssLabels, lbl, ',') && lblIndex < n) {
        strncpy(labels[lblIndex], lbl.c_str(), 19);
        labels[lblIndex][19] = 0;
        lblIndex++;
    }
    if (lblIndex != n) return false;

    // Read values line (comma separated)
    if (!std::getline(inFile, line)) return false;
    std::istringstream ssVals(line);
    std::string valToken;
    int valIndex = 0;
    while (std::getline(ssVals, valToken, ',') && valIndex < n) {
        values[valIndex++] = std::stoi(valToken);
    }
    if (valIndex != n) return false;

    // If stacked bar chart, read stackedValues n lines
    if (currentChartType == 9) {
        for (int i = 0; i < n; i++) {
            if (!std::getline(inFile, line)) return false;
            std::istringstream ssStack(line);
            std::string stkVal;
            int stacks[3] = {0,0,0};
            int k = 0;
            while (std::getline(ssStack, stkVal, ',') && k < 3) {
                stacks[k] = std::stoi(stkVal);
                k++;
            }
            if (k != 3) return false;
            for (k = 0; k < 3; k++) stackedValues[i][k] = stacks[k];
        }
    }
    inFile.close();
    return true;
}

// showLoadOrNewDataScreen implementation
bool showLoadOrNewDataScreen(int centerX) {
    using namespace std;

    cleardevice();
    setbkcolor(WHITE);
    cleardevice();

    setcolor(BLACK);
    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
    outtextxy(centerX - 180, 30, (char*)"Load Saved Data or Enter New");

    // List files
    vector<string> files = listSavedFiles(saveFolderPath);

    int btnWidth = 400, btnHeight = 40, spacing = 10;
    int startY = 100;

    if (files.empty()) {
        setcolor(RED);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
        outtextxy(centerX - 150, getmaxy() / 2, (char*)"No saved files found.");
        delay(1500);
        return true;   // no saved - must enter new
    }

    // Draw buttons for files
    for (int i = 0; i < (int)files.size(); i++) {
        int ypos = startY + i * (btnHeight + spacing);
        setfillstyle(SOLID_FILL, LIGHTCYAN);
        bar(centerX - btnWidth / 2, ypos, centerX + btnWidth / 2, ypos + btnHeight);
        setcolor(BLACK);
        rectangle(centerX - btnWidth / 2, ypos, centerX + btnWidth / 2, ypos + btnHeight);
        outtextxy(centerX - btnWidth / 2 + 10, ypos + 10, (char*)files[i].c_str());
    }

    // Buttons: Enter New Data, Exit
    int btnNewY = startY + (int)files.size() * (btnHeight + spacing) + 30;
    int btnExitY = btnNewY + btnHeight + 20;
    const char* btnNewText = "Enter New Data";
    const char* btnExitText = "Exit";

    setfillstyle(SOLID_FILL, LIGHTGREEN);
    bar(centerX - btnWidth / 2, btnNewY, centerX + btnWidth / 2, btnNewY + btnHeight);
    setcolor(BLACK);
    rectangle(centerX - btnWidth / 2, btnNewY, centerX + btnWidth / 2, btnNewY + btnHeight);
    outtextxy(centerX - btnWidth / 2 + 10, btnNewY + 10, (char*)btnNewText);

    setfillstyle(SOLID_FILL, LIGHTRED);
    bar(centerX - btnWidth / 2, btnExitY, centerX + btnWidth / 2, btnExitY + btnHeight);
    setcolor(BLACK);
    rectangle(centerX - btnWidth / 2, btnExitY, centerX + btnWidth / 2, btnExitY + btnHeight);
    outtextxy(centerX - btnWidth / 2 + 10, btnExitY + 10, (char*)btnExitText);

    int mx, my;
    while (true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);
            // Check clicked on file buttons:
            for (int i = 0; i < (int)files.size(); i++) {
                int ypos = startY + i * (btnHeight + spacing);
                if (mx >= centerX - btnWidth / 2 && mx <= centerX + btnWidth / 2 &&
                    my >= ypos && my <= ypos + btnHeight) {
                    clearmouseclick(WM_LBUTTONDOWN);
                    bool loaded = loadChartFromFile(saveFolderPath + files[i]);
                    if (loaded) {
                        return false;  // loaded saved data, proceed
                    }
                    else {
                        // show error message and continue loop
                        cleardevice();
                        setcolor(RED);
                        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
                        outtextxy(centerX - 150, getmaxy() / 2, (char*)"Failed to load file.");
                        delay(1500);
                        return true;  // fail to load, enter new data
                    }
                }
            }

            // Check enter new
            if (mx >= centerX - btnWidth / 2 && mx <= centerX + btnWidth / 2) {
                if (my >= btnNewY && my <= btnNewY + btnHeight) {
                    clearmouseclick(WM_LBUTTONDOWN);
                    return true;  // enter new data
                }
                // Check exit
                else if (my >= btnExitY && my <= btnExitY + btnHeight) {
                    clearmouseclick(WM_LBUTTONDOWN);
                    exit(0);  // Or set flag to exit program if you want
                }
            }
            clearmouseclick(WM_LBUTTONDOWN);
        }
        delay(50);
    }
    return true;
}
void getChartData(int centerX) {
    cleardevice();
    setbkcolor(WHITE);
    cleardevice();

    int mx, my;
    char input[4] = "", ch;
    int len = 0;
    int boxX = 420, boxY = 95, boxW = 100, boxH = 30;

    // Draw header text
    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
    setcolor(BLACK);
    outtextxy(100, 20, "Data Input Screen");

    // Display message about stacked bar data input location, with spacing
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 1);
    outtextxy(100, 60, "NOTE: To enter data for Stacked Bar Chart,");
    outtextxy(100, 80, "please select 'Stacked Bar Chart' in chart selection first.");

    // Button to return to chart selection
    int btnX = getmaxx() - 350;   // Move button to the right side
int btnY = 100;               // Vertically align near top, adjust as needed
int btnW = 300;
int btnH = 40;
    setfillstyle(SOLID_FILL, MAGENTA);
    bar(btnX, btnY, btnX + btnW, btnY + btnH);
    setcolor(BLACK);
    rectangle(btnX, btnY, btnX + btnW, btnY + btnH);
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
    outtextxy(btnX + 10, btnY + 7, "Go to Chart Selection");

    // Draw input prompt box for number of bars
    setcolor(RED);
    rectangle(boxX, boxY, boxX + boxW, boxY + boxH);
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
    outtextxy(100, 100, "Enter number of bars (max 10):");
    outtextxy(100, 140, "Click inside the box to type...");

    // Wait for click on either input box or return button
    while(true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);
            // Return button click
          if (mx >= btnX && mx <= btnX + btnW && my >= btnY && my <= btnY + btnH) {
    clearmouseclick(WM_LBUTTONDOWN);
    return;  // Go back to chart selection
}
            // Input box click
            else if(mx >= boxX && mx <= boxX + boxW && my >= boxY && my <= boxY + boxH) {
                clearmouseclick(WM_LBUTTONDOWN);
                break;
            }
            else {
                clearmouseclick(WM_LBUTTONDOWN);
            }
        }
        delay(50);
    }

    // Input number
    len = 0;
    input[0] = 0;
    while ((ch = getch()) != 13) {
        if (ch == 8 && len > 0) input[--len] = '\0';
        else if ((ch >= '0' && ch <= '9') && len < 2) input[len++] = ch, input[len] = '\0';
        setfillstyle(SOLID_FILL, WHITE);
        bar(boxX + 1, boxY + 1, boxX + boxW - 1, boxY + boxH - 1);
        rectangle(boxX, boxY, boxX + boxW, boxY + boxH);
        outtextxy(boxX + 5, boxY + 5, input);
    }

    n = atoi(input);
    if (n <= 0 || n > 10) {
        // show error message
        setcolor(RED);
        bar(50, 180, getmaxx() - 50, 230);
        outtextxy(100, 200, "Please enter a valid number between 1 and 10.");
        delay(1500);
        getChartData(centerX);  // restart input
        return;
    }

    // For each bar, get label and value
    for (int i = 0; i < n; i++) {
        char lbl[20] = "", val[6] = "";
        len = 0;

        // Clear relevant area before input
        setfillstyle(SOLID_FILL, WHITE);
        bar(100, 180, 700, 310);

        // Label input prompt
        outtextxy(100, 180, "Label for bar ");
        char idx[4]; itoa(i + 1, idx, 10);
        outtextxy(240, 180, idx);

        while ((ch = getch()) != 13) {
            if (ch == 8 && len > 0) lbl[--len] = '\0';
            else if (len < 18 && ch >= 32 && ch <= 126) lbl[len++] = ch, lbl[len] = '\0';
            setfillstyle(SOLID_FILL, WHITE);
            bar(100, 210, 700, 240);
            outtextxy(100, 210, lbl);
        }
        strcpy(labels[i], lbl);

        // Value input prompt
        outtextxy(100, 250, "Value for this bar:");
        len = 0;
        memset(val, 0, sizeof(val));
        while ((ch = getch()) != 13) {
            if (ch == 8 && len > 0) val[--len] = '\0';
            else if (len < 5 && ch >= '0' && ch <= '9') val[len++] = ch, val[len] = '\0';
            setfillstyle(SOLID_FILL, WHITE);
            bar(320, 250, 420, 280);
            outtextxy(320, 250, val);
        }
        values[i] = atoi(val);
    }
}

// Stacked Bar data input: new with proper messages, spacing, return button
void getStackedBarData(int centerX) {
    cleardevice();
    setbkcolor(WHITE);
    cleardevice();

    int mx, my;
    char input[4] = "", ch;
    int len = 0;
    int boxX = 420, boxY = 95, boxW = 100, boxH = 30;

    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
    setcolor(BLACK);
    outtextxy(50, 20, "Stacked Bar Chart Data Input");

    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 1);
    outtextxy(50, 60, "Enter number of bars (max 10):");

    // Return button
    int btnX = 100, btnY = 400, btnW = 360, btnH = 40;
    setfillstyle(SOLID_FILL, LIGHTCYAN);
    bar(btnX, btnY, btnX + btnW, btnY + btnH);
    setcolor(BLACK);
    rectangle(btnX, btnY, btnX + btnW, btnY + btnH);
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
    outtextxy(btnX + 15, btnY + 10, "Return to Chart Selection");

    // Input box for number of bars
    setcolor(RED);
    rectangle(boxX, boxY, boxX + boxW, boxY + boxH);
    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
    outtextxy(100, 140, "Click inside the box to type...");

    while (true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);
            if (mx >= btnX && mx <= btnX + btnW && my >= btnY && my <= btnY + btnH) {
                clearmouseclick(WM_LBUTTONDOWN);
                // User clicks return button, go back to chart selection
                return;
            }
            if (mx >= boxX && mx <= boxX + boxW && my >= boxY && my <= boxY + boxH) {
                clearmouseclick(WM_LBUTTONDOWN);
                break;
            }
            clearmouseclick(WM_LBUTTONDOWN);
        }
        delay(50);
    }

    len = 0;
    input[0] = 0;

    while ((ch = getch()) != 13) {
        if (ch == 8 && len > 0) input[--len] = '\0';
        else if ((ch >= '0' && ch <= '9') && len < 2) input[len++] = ch, input[len] = '\0';

        setfillstyle(SOLID_FILL, WHITE);
        bar(boxX + 1, boxY + 1, boxX + boxW - 1, boxY + boxH - 1);
        rectangle(boxX, boxY, boxX + boxW, boxY + boxH);
        outtextxy(boxX + 5, boxY + 5, input);
    }

    n = atoi(input);
    if (n <= 0 || n > 10) {
        setcolor(RED);
        bar(50, 180, getmaxx() - 50, 230);
        outtextxy(100, 200, "Please enter a valid number between 1 and 10.");
        delay(1500);
        getStackedBarData(centerX);
        return;
    }

    for (int i = 0; i < n; i++) {
        char lbl[20] = "", val[6] = "";
        int valStacks[3] = {0, 0, 0};
        int lenLbl = 0, lenVal;

        setfillstyle(SOLID_FILL, WHITE);
        bar(100, 180, 700, 380);

        outtextxy(100, 180, "Label for Bar ");
        char idx[4];
        itoa(i + 1, idx, 10);
        outtextxy(260, 180, idx);

        while ((ch = getch()) != 13) {
            if (ch == 8 && lenLbl > 0) lbl[--lenLbl] = '\0';
            else if (lenLbl < 18 && ch >= 32 && ch <= 126) lbl[lenLbl++] = ch, lbl[lenLbl] = '\0';
            setfillstyle(SOLID_FILL, WHITE);
            bar(100, 210, 700, 240);
            outtextxy(100, 210, lbl);
        }
        strcpy(labels[i], lbl);

        // Input stack values with spacing and labels
        for (int st = 0; st < 3; st++) {
            lenVal = 0;
            setfillstyle(SOLID_FILL, WHITE);
            int yStart = 250 + st * 40;
            bar(320, yStart, 420, yStart + 30);
            char prompt[40];
            sprintf(prompt, "Value for stack %d:", st + 1);
            outtextxy(100, yStart, prompt);

            while ((ch = getch()) != 13) {
                if (ch == 8 && lenVal > 0) val[--lenVal] = '\0';
                else if (lenVal < 5 && ch >= '0' && ch <= '9') val[lenVal++] = ch, val[lenVal] = '\0';
                setfillstyle(SOLID_FILL, WHITE);
                bar(320, yStart, 420, yStart + 30);
                outtextxy(320, yStart, val);
            }
            valStacks[st] = atoi(val);
            val[0] = '\0';
        }
        for (int k = 0; k < 3; k++) stackedValues[i][k] = valStacks[k];
        values[i] = valStacks[0] + valStacks[1] + valStacks[2];
    }
}

// Menu for charts

// Menu for charts
int showChartMenu(int centerX) {
    const char* chartOptions[] = {
        "1. Bar Chart", "2. Pie Chart", "3. Line Chart", "4. Donut Chart",
        "5. Scatter Plot", "6. Area Chart", "7. Column Chart",
        "8. Histogram Chart", "9. Stacked Bar Chart", "10. Multi-Line Chart"
    };

    int buttonWidth = 320;
    int buttonHeight = 50;
    int spacing = 10;

    int startX = centerX - buttonWidth / 2;
    int startY = 100;

    settextstyle(TRIPLEX_FONT, HORIZ_DIR, 4);
    setcolor(BLUE);
    outtextxy(centerX - textwidth("Choose Chart Type") / 2, 30, "Choose Chart Type");

    settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
    for (int i = 0; i < 10; i++) {
        int boxY = startY + i * (buttonHeight + spacing);

        setfillstyle(SOLID_FILL, LIGHTCYAN);
        bar(startX, boxY, startX + buttonWidth, boxY + buttonHeight);

        setcolor(BLACK);
        rectangle(startX, boxY, startX + buttonWidth, boxY + buttonHeight);

        int textY = boxY + (buttonHeight - textheight("A")) / 2;
        outtextxy(startX + 10, textY, (char*)chartOptions[i]);
    }

    int mx, my;
    while (true) {
        if (ismouseclick(WM_LBUTTONDOWN)) {
            getmouseclick(WM_LBUTTONDOWN, mx, my);
            for (int i = 0; i < 10; i++) {
                int boxY = startY + i * (buttonHeight + spacing);
                if (mx >= startX && mx <= startX + buttonWidth &&
                    my >= boxY && my <= boxY + buttonHeight)
                    return i + 1;
            }
        }
        delay(50);
    }
}

int main() {
    int width = GetSystemMetrics(SM_CXSCREEN);
    int height = GetSystemMetrics(SM_CYSCREEN);
    initwindow(width, height, "CHART VISUALIZER", 0, 0);

    int centerX = width / 2;
    saveFolderPath = getSaveFolderPath();
    bool exitProgram = false;
    while (!exitProgram) {        // Outer loop to allow returning to welcome screen
        setbkcolor(WHITE);
        cleardevice();

        // Welcome screen
        setcolor(BLUE);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 6);
        int textWidth = textwidth((char*)"CHART VISUALIZER");
        outtextxy(centerX - textWidth / 2, 100, (char*)"CHART VISUALIZER");
        setcolor(DARKGRAY);
        settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
        const char* quote = "\"Visualizing data is the first step to understanding it.\"";
        outtextxy(centerX - (strlen(quote) * 8) / 2, 180, (char*)quote);

        int chartY = 260;
        int startX = centerX - 160;
        setfillstyle(SOLID_FILL, LIGHTBLUE); bar(startX, chartY - 30, startX + 20, chartY);
        setfillstyle(SOLID_FILL, LIGHTGREEN); bar(startX + 30, chartY - 50, startX + 50, chartY);
        setfillstyle(SOLID_FILL, LIGHTRED); bar(startX + 60, chartY - 20, startX + 80, chartY);
        setcolor(BLACK);
        setfillstyle(SOLID_FILL, CYAN); pieslice(centerX, chartY, 0, 90, 30);
        setfillstyle(SOLID_FILL, MAGENTA); pieslice(centerX, chartY, 90, 180, 30);
        setfillstyle(SOLID_FILL, YELLOW); pieslice(centerX, chartY, 180, 360, 30);
        setcolor(RED);
        int lx = centerX + 100;
        line(lx, chartY - 10, lx + 20, chartY - 30);
        line(lx + 20, chartY - 30, lx + 40, chartY - 15);
        line(lx + 40, chartY - 15, lx + 60, chartY - 35);
        int baseY = getmaxy() - 220;
        setcolor(CYAN);
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 2);
        outtextxy(getmaxx() - 380, baseY, (char*)"Group Members:");
        outtextxy(getmaxx() - 480, baseY + 25, (char*)"- Khadija Ikram (24251101027)");
        outtextxy(getmaxx() - 480, baseY + 50, (char*)"- Ayesha Afzal (24251101009)");
        outtextxy(getmaxx() - 480, baseY + 75, (char*)"- Amna Faryad (24251101005)");
        settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 3);
        outtextxy(50, baseY + 100, (char*)"Supervised by: Dr. Mariam Nosheen");
        settextstyle(TRIPLEX_FONT, HORIZ_DIR, 3);
        outtextxy(centerX - 100, baseY + 60, (char*)"OOP Project");
        settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
        outtextxy(centerX - 70, baseY + 85, (char*)"BSCS - Semester 2");
        outtextxy(centerX - 40, baseY + 110, (char*)"LCWU");
        setcolor(BLUE);
        outtextxy(centerX - 150, getmaxy() - 50, (char*)"Click to continue...");
        while (!ismouseclick(WM_LBUTTONDOWN)) delay(100);
        clearmouseclick(WM_LBUTTONDOWN);

        bool enterNewData = showLoadOrNewDataScreen(centerX);

        if (enterNewData) {
            getChartData(centerX);
        }

        bool backToWelcome = false;
        while (!backToWelcome) {
            cleardevice();
            setbkcolor(WHITE);

            int  choice;
			choice = showChartMenu(centerX);

            currentChartType = choice;

            if (choice == 9) {
                getStackedBarData(centerX);
            }

            Chart* chart = nullptr;
            switch (choice) {
                case 1: chart = new BarChart(); break;
                case 2: chart = new PieChart(); break;
                case 3: chart = new LineChart(); break;
                case 4: chart = new DonutChart(); break;
                case 5: chart = new ScatterPlot(); break;
                case 6: chart = new AreaChart(); break;
                case 7: chart = new ColumnChart(); break;
                case 8: chart = new HistogramChart(); break;
                case 9: chart = new StackedBarChart(); break;
                case 10: chart = new MultiLineChart(); break;
                default: continue;
            }

            int nav = chart->draw(centerX);

            backToWelcome = handleNav(nav, centerX);

            delete chart;

            if (backToWelcome) break;
        }
    }

    closegraph();
    return 0;
}
