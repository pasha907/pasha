import tkinter as tk
import math

W, H = 1000, 650
N = 46          # number of body segments
SEG = 13        # distance between segments

root = tk.Tk()
root.title("Dragon follows your cursor")
canvas = tk.Canvas(root, width=W, height=H, bg="#0b0f1a", highlightthickness=0)
canvas.pack()

pts = [[W / 2 - i * SEG, H / 2] for i in range(N)]
mouse = [W / 2, H / 2]
t = 0


def on_move(e):
    mouse[0], mouse[1] = e.x, e.y


canvas.bind("<Motion>", on_move)


def d(angle, length):
    return length * math.cos(angle), length * math.sin(angle)


def heading(i):
    a = pts[max(i - 1, 0)]
    b = pts[min(i + 1, N - 1)]
    return math.atan2(a[1] - b[1], a[0] - b[0])


def update():
    global t
    t += 1

    # head eases toward the cursor
    dx, dy = mouse[0] - pts[0][0], mouse[1] - pts[0][1]
    dist = math.hypot(dx, dy)
    if dist > 3:
        speed = min(dist * 0.12, 11)
        pts[0][0] += dx / dist * speed
        pts[0][1] += dy / dist * speed

    # every segment follows the one in front
    for i in range(1, N):
        dx, dy = pts[i - 1][0] - pts[i][0], pts[i - 1][1] - pts[i][1]
        dist = math.hypot(dx, dy) or 1
        pts[i][0] = pts[i - 1][0] - dx / dist * SEG
        pts[i][1] = pts[i - 1][1] - dy / dist * SEG

    draw()
    root.after(16, update)


def draw():
    canvas.delete("all")
    flap = math.sin(t * 0.25) * 0.35

    # target glow
    canvas.create_oval(mouse[0] - 6, mouse[1] - 6, mouse[0] + 6, mouse[1] + 6,
                       outline="#ffcc33", width=2)

    # legs (behind body)
    for i in (11, 27):
        h = heading(i)
        for s in (-1, 1):
            fx, fy = d(h + s * 1.1, 26)
            canvas.create_line(pts[i][0], pts[i][1], pts[i][0] + fx, pts[i][1] + fy,
                               fill="#1f7a4d", width=5, capstyle="round")

    # wings
    base = pts[6]
    h = heading(6)
    for s in (-1, 1):
        poly = [base[0], base[1]]
        for ang, ln in ((1.3, 70), (2.0, 100), (2.7, 65)):
            ox, oy = d(h + s * (ang - flap * 1.5), ln)
            poly += [base[0] + ox, base[1] + oy]
        canvas.create_polygon(poly, fill="#7a1fa2", outline="#c77dff", width=2)

    # body, tail to head
    for i in range(N - 1, -1, -1):
        r = max(2, 14 - i * 0.26) if i > 0 else 14
        shade = int(90 + 100 * (1 - i / N))
        color = f"#{20:02x}{shade:02x}{80:02x}"
        x, y = pts[i]
        canvas.create_oval(x - r, y - r, x + r, y + r, fill=color, outline="#0f4d30")
        if i % 2 == 0 and i > 1:      # back spikes
            h = heading(i)
            sx, sy = d(h + math.pi / 2, r + 5)
            canvas.create_line(x, y, x + sx, y + sy, fill="#ff5d5d", width=2)

    # head
    x, y = pts[0]
    h = heading(0)
    nx, ny = d(h, 24)
    canvas.create_polygon(x + nx, y + ny,
                          x + d(h + 1.6, 13)[0], y + d(h + 1.6, 13)[1],
                          x + d(h + math.pi, 6)[0], y + d(h + math.pi, 6)[1],
                          x + d(h - 1.6, 13)[0], y + d(h - 1.6, 13)[1],
                          fill="#2aa866", outline="#0f4d30")
    for s in (-1, 1):
        # horns
        hx, hy = x + d(h + s * 1.3, 8)[0], y + d(h + s * 1.3, 8)[1]
        ex, ey = hx + d(h + s * 2.7, 24)[0], hy + d(h + s * 2.7, 24)[1]
        canvas.create_line(hx, hy, ex, ey, fill="#f5e6a8", width=4, capstyle="round")
        # eyes
        ax, ay = x + d(h + s * 0.8, 9)[0] + d(h, 5)[0], y + d(h + s * 0.8, 9)[1] + d(h, 5)[1]
        canvas.create_oval(ax - 3, ay - 3, ax + 3, ay + 3, fill="#ffdd33", outline="")


update()
root.mainloop()
