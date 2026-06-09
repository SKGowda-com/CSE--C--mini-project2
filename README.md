# CSE--C--mini-project2#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

/* ─── Canvas dimensions ─── */
#define ROWS 40
#define COLS 80
#define MAX_OBJECTS 50

/* ─── Character used for drawing and blank space ─── */
#define DRAW_CHAR '*'
#define BLANK_CHAR ' '
#define FLOOR_CHAR '_'

/* ─── Object types ─── */
#define OBJ_LINE      1
#define OBJ_RECTANGLE 2
#define OBJ_TRIANGLE  3
#define OBJ_CIRCLE    4

/* ─── Structure to remember each drawn object ─── */
typedef struct {
    int type;           /* OBJ_LINE, OBJ_RECTANGLE, etc. */
    int id;             /* unique id for deletion         */
    /* Generic params – meaning depends on type:
       LINE      : p[0]=x1, p[1]=y1, p[2]=x2, p[3]=y2
       RECTANGLE : p[0]=top-left x, p[1]=top-left y, p[2]=width, p[3]=height
       TRIANGLE  : p[0]=x1,p[1]=y1, p[2]=x2,p[3]=y2, p[4]=x3,p[5]=y3
       CIRCLE    : p[0]=cx, p[1]=cy, p[2]=radius                            */
    int p[6];
} Object;

/* ─── Globals ─── */
char canvas[ROWS][COLS];
Object objects[MAX_OBJECTS];
int objectCount = 0;
int nextId = 1;

/* ═══════════════════════════════════════════════════════════════
   Utility helpers
   ═══════════════════════════════════════════════════════════════ */

/* Clear the canvas to blank */
void clearCanvas(void) {
    int r, c;
    for (r = 0; r < ROWS; r++)
        for (c = 0; c < COLS; c++)
            canvas[r][c] = BLANK_CHAR;
}

/* Safely set a pixel on the canvas */
void setPixel(int r, int c, char ch) {
    if (r >= 0 && r < ROWS && c >= 0 && c < COLS)
        canvas[r][c] = ch;
}

/* ═══════════════════════════════════════════════════════════════
   Drawing primitives
   ═══════════════════════════════════════════════════════════════ */

/* Draw a line using Bresenham's algorithm.
   Uses '*' for sloped segments and '_' for horizontal segments. */
void drawLine(int r1, int c1, int r2, int c2) {
    int dr = abs(r2 - r1);
    int dc = abs(c2 - c1);
    int sr = (r1 < r2) ? 1 : -1;
    int sc = (c1 < c2) ? 1 : -1;
    int err = dr - dc;
    int e2;
    int isHorizontal = (r1 == r2);

    while (1) {
        setPixel(r1, c1, isHorizontal ? FLOOR_CHAR : DRAW_CHAR);
        if (r1 == r2 && c1 == c2) break;
        e2 = 2 * err;
        if (e2 > -dc) { err -= dc; r1 += sr; }
        if (e2 <  dr) { err += dr; c1 += sc; }
    }
}

/* Draw a rectangle.
   Top and bottom edges use '_', left and right edges use '*'. */
void drawRectangle(int topR, int topC, int width, int height) {
    int r, c;
    int botR = topR + height - 1;
    int rightC = topC + width - 1;

    /* Top edge */
    for (c = topC; c <= rightC; c++)
        setPixel(topR, c, FLOOR_CHAR);

    /* Bottom edge */
    for (c = topC; c <= rightC; c++)
        setPixel(botR, c, FLOOR_CHAR);

    /* Left edge */
    for (r = topR; r <= botR; r++)
        setPixel(r, topC, DRAW_CHAR);

    /* Right edge */
    for (r = topR; r <= botR; r++)
        setPixel(r, rightC, DRAW_CHAR);

    /* Corners always '*' */
    setPixel(topR, topC, DRAW_CHAR);
    setPixel(topR, rightC, DRAW_CHAR);
    setPixel(botR, topC, DRAW_CHAR);
    setPixel(botR, rightC, DRAW_CHAR);
}

/* Draw a triangle by connecting three vertices with lines. */
void drawTriangle(int r1, int c1, int r2, int c2, int r3, int c3) {
    drawLine(r1, c1, r2, c2);
    drawLine(r2, c2, r3, c3);
    drawLine(r3, c3, r1, c1);
}

/* Draw a circle using the midpoint circle algorithm.
   Uses '_' for the top and bottom arcs, '*' elsewhere. */
void drawCircle(int cr, int cc, int radius) {
    int x = 0, y = radius;
    int d = 1 - radius;

    while (x <= y) {
        /* Use '_' when the point is near the top or bottom of the circle */
        setPixel(cr - y, cc + x, FLOOR_CHAR);   /* top arc */
        setPixel(cr - y, cc - x, FLOOR_CHAR);   /* top arc */
        setPixel(cr + y, cc + x, FLOOR_CHAR);   /* bottom arc */
        setPixel(cr + y, cc - x, FLOOR_CHAR);   /* bottom arc */

        setPixel(cr - x, cc + y, DRAW_CHAR);    /* right side */
        setPixel(cr - x, cc - y, DRAW_CHAR);    /* left side */
        setPixel(cr + x, cc + y, DRAW_CHAR);    /* right side */
        setPixel(cr + x, cc - y, DRAW_CHAR);    /* left side */

        if (d < 0) {
            d += 2 * x + 3;
        } else {
            d += 2 * (x - y) + 5;
            y--;
        }
        x++;
    }
}

/* ═══════════════════════════════════════════════════════════════
   Redraw everything from the object list
   ═══════════════════════════════════════════════════════════════ */
void redrawAll(void) {
    int i;
    clearCanvas();
    for (i = 0; i < objectCount; i++) {
        Object *o = &objects[i];
        switch (o->type) {
            case OBJ_LINE:
                drawLine(o->p[0], o->p[1], o->p[2], o->p[3]);
                break;
            case OBJ_RECTANGLE:
                drawRectangle(o->p[0], o->p[1], o->p[2], o->p[3]);
                break;
            case OBJ_TRIANGLE:
                drawTriangle(o->p[0], o->p[1], o->p[2], o->p[3], o->p[4], o->p[5]);
                break;
            case OBJ_CIRCLE:
                drawCircle(o->p[0], o->p[1], o->p[2]);
                break;
        }
    }
}

/* ═══════════════════════════════════════════════════════════════
   Display the picture
   ═══════════════════════════════════════════════════════════════ */
void displayPicture(void) {
    int r, c;

    /* Top border */
    printf("+");
    for (c = 0; c < COLS; c++) printf("-");
    printf("+\n");

    for (r = 0; r < ROWS; r++) {
        printf("|");
        for (c = 0; c < COLS; c++)
            putchar(canvas[r][c]);
        printf("|\n");
    }

    /* Bottom border */
    printf("+");
    for (c = 0; c < COLS; c++) printf("-");
    printf("+\n");
}

/* ═══════════════════════════════════════════════════════════════
   List all objects currently in the picture
   ═══════════════════════════════════════════════════════════════ */
void listObjects(void) {
    int i;
    if (objectCount == 0) {
        printf("  (no objects on canvas)\n");
        return;
    }
    printf("  %-4s  %-12s  %s\n", "ID", "Type", "Parameters");
    printf("  ----  ------------  -----------------------------------\n");
    for (i = 0; i < objectCount; i++) {
        Object *o = &objects[i];
        printf("  %-4d  ", o->id);
        switch (o->type) {
            case OBJ_LINE:
                printf("%-12s  (%d,%d) -> (%d,%d)\n", "Line",
                       o->p[0], o->p[1], o->p[2], o->p[3]);
                break;
            case OBJ_RECTANGLE:
                printf("%-12s  top-left(%d,%d)  w=%d  h=%d\n", "Rectangle",
                       o->p[0], o->p[1], o->p[2], o->p[3]);
                break;
            case OBJ_TRIANGLE:
                printf("%-12s  (%d,%d) (%d,%d) (%d,%d)\n", "Triangle",
                       o->p[0], o->p[1], o->p[2], o->p[3], o->p[4], o->p[5]);
                break;
            case OBJ_CIRCLE:
                printf("%-12s  center(%d,%d)  r=%d\n", "Circle",
                       o->p[0], o->p[1], o->p[2]);
                break;
        }
    }
}

/* ═══════════════════════════════════════════════════════════════
   Add-object helpers  (return 1 on success)
   ═══════════════════════════════════════════════════════════════ */

int addLine(void) {
    int r1, c1, r2, c2;
    if (objectCount >= MAX_OBJECTS) { printf("  Object limit reached!\n"); return 0; }
    printf("  Enter start point (row col): ");
    scanf("%d %d", &r1, &c1);
    printf("  Enter end point   (row col): ");
    scanf("%d %d", &r2, &c2);

    Object *o = &objects[objectCount++];
    o->type = OBJ_LINE;
    o->id   = nextId++;
    o->p[0] = r1; o->p[1] = c1;
    o->p[2] = r2; o->p[3] = c2;

    redrawAll();
    printf("  Line added (id %d).\n", o->id);
    return 1;
}

int addRectangle(void) {
    int r, c, w, h;
    if (objectCount >= MAX_OBJECTS) { printf("  Object limit reached!\n"); return 0; }
    printf("  Enter top-left corner (row col): ");
    scanf("%d %d", &r, &c);
    printf("  Enter width and height: ");
    scanf("%d %d", &w, &h);
    if (w < 2 || h < 2) { printf("  Width and height must be >= 2.\n"); return 0; }

    Object *o = &objects[objectCount++];
    o->type = OBJ_RECTANGLE;
    o->id   = nextId++;
    o->p[0] = r; o->p[1] = c;
    o->p[2] = w; o->p[3] = h;

    redrawAll();
    printf("  Rectangle added (id %d).\n", o->id);
    return 1;
}

int addTriangle(void) {
    int r1, c1, r2, c2, r3, c3;
    if (objectCount >= MAX_OBJECTS) { printf("  Object limit reached!\n"); return 0; }
    printf("  Enter vertex 1 (row col): ");
    scanf("%d %d", &r1, &c1);
    printf("  Enter vertex 2 (row col): ");
    scanf("%d %d", &r2, &c2);
    printf("  Enter vertex 3 (row col): ");
    scanf("%d %d", &r3, &c3);

    Object *o = &objects[objectCount++];
    o->type = OBJ_TRIANGLE;
    o->id   = nextId++;
    o->p[0] = r1; o->p[1] = c1;
    o->p[2] = r2; o->p[3] = c2;
    o->p[4] = r3; o->p[5] = c3;

    redrawAll();
    printf("  Triangle added (id %d).\n", o->id);
    return 1;
}

int addCircle(void) {
    int cr, cc, rad;
    if (objectCount >= MAX_OBJECTS) { printf("  Object limit reached!\n"); return 0; }
    printf("  Enter center (row col): ");
    scanf("%d %d", &cr, &cc);
    printf("  Enter radius: ");
    scanf("%d", &rad);
    if (rad < 1) { printf("  Radius must be >= 1.\n"); return 0; }

    Object *o = &objects[objectCount++];
    o->type = OBJ_CIRCLE;
    o->id   = nextId++;
    o->p[0] = cr; o->p[1] = cc;
    o->p[2] = rad;

    redrawAll();
    printf("  Circle added (id %d).\n", o->id);
    return 1;
}

/* ═══════════════════════════════════════════════════════════════
   Delete an object by id
   ═══════════════════════════════════════════════════════════════ */
int deleteObject(void) {
    int id, i, found = -1;
    if (objectCount == 0) { printf("  Nothing to delete.\n"); return 0; }

    printf("\n  Current objects:\n");
    listObjects();
    printf("\n  Enter object ID to delete: ");
    scanf("%d", &id);

    for (i = 0; i < objectCount; i++) {
        if (objects[i].id == id) { found = i; break; }
    }
    if (found == -1) { printf("  Object with ID %d not found.\n", id); return 0; }

    /* Shift remaining objects down */
    for (i = found; i < objectCount - 1; i++)
        objects[i] = objects[i + 1];
    objectCount--;

    redrawAll();
    printf("  Object %d deleted.\n", id);
    return 1;
}

/* ═══════════════════════════════════════════════════════════════
   Clear all objects
   ═══════════════════════════════════════════════════════════════ */
void clearAll(void) {
    objectCount = 0;
    clearCanvas();
    printf("  Canvas cleared.\n");
}

/* ═══════════════════════════════════════════════════════════════
   Main menu
   ═══════════════════════════════════════════════════════════════ */
int main(void) {
    int choice;
    clearCanvas();

    printf("\n");
    printf("  ====================================\n");
    printf("    2-D  Graphics  Editor  (C + ASCII)\n");
    printf("    Canvas: %d rows x %d cols\n", ROWS, COLS);
    printf("    Characters: * (edges)  _ (flats)\n");
    printf("  ====================================\n");

    while (1) {
        printf("\n  ┌─────────────── MENU ───────────────┐\n");
        printf("  │  1. Add Line                        │\n");
        printf("  │  2. Add Rectangle                   │\n");
        printf("  │  3. Add Triangle                    │\n");
        printf("  │  4. Add Circle                      │\n");
        printf("  │  5. Delete Object                   │\n");
        printf("  │  6. Display Picture                 │\n");
        printf("  │  7. List Objects                    │\n");
        printf("  │  8. Clear All                       │\n");
        printf("  │  0. Exit                            │\n");
        printf("  └─────────────────────────────────────┘\n");
        printf("  Enter choice: ");
        scanf("%d", &choice);
        printf("\n");

        switch (choice) {
            case 1: addLine();      break;
            case 2: addRectangle(); break;
            case 3: addTriangle();  break;
            case 4: addCircle();    break;
            case 5: deleteObject(); break;
            case 6: displayPicture(); break;
            case 7: listObjects();  break;
            case 8: clearAll();     break;
            case 0:
                printf("  Goodbye!\n");
                return 0;
            default:
                printf("  Invalid choice. Try again.\n");
        }
    }

    return 0;
}
