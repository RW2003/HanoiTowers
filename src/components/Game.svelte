<script lang="ts">
    import { onMount } from "svelte";

    let container: HTMLDivElement;
    let rings: Array<HTMLElement> = [];
    let targets = [] as any[];
    let rings_currentPos = {} as any;

    // Reichweite in Pixeln, ab wann das Element einrasten soll
    const SNAP_THRESHOLD = 500; 
    const RING_COUNT     = 8; 
    const STEP_TIMEOUT   = 200; 

    let isDragging = false;
    let locked = false;
    let startX = 0;
    let startY = 0;

    let resetX = '0px';
    let resetY = '0px';


    const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

    onMount(() => {
        reset();
    });


    function event_pointerdown(e: Event) {
        let draggable  = e.target as HTMLElement;
        let currentPos = rings_currentPos[draggable?.id];

        if(!draggable || !currentPos || locked)
            return;

        if(draggable.snappedTo && draggable.snappedIndex < (draggable.snappedTo.count - 1))
            return;

        let currentX = currentPos.x;
        let currentY = currentPos.y;

        isDragging = true;
        draggable.classList.remove('snapping');
        
        // Klickposition relativ zur oberen linken Ecke des Elements berechnen
        startX = e.clientX - currentX;
        startY = e.clientY - currentY;

        resetX = draggable.style.left;
        resetY = draggable.style.top;
        
        draggable.setPointerCapture(e.pointerId); // Bindet Maus/Touch fest an das Element
    }

    function event_pointermove(e: Event) {
        let draggable = e.target as HTMLElement;

        if(!draggable || locked)
            return;

        if (!isDragging)
            return;

        // Neue Position berechnen
        let currentX = e.clientX - startX;
        let currentY = e.clientY - startY;

        // Optionale Begrenzung (Boundary) innerhalb des Containers
        const maxW = container.clientWidth - draggable.clientWidth;
        const maxH = container.clientHeight - draggable.clientHeight;
        currentX = Math.max(0, Math.min(currentX, maxW));
        currentY = Math.max(0, Math.min(currentY, maxH));

        rings_currentPos[draggable?.id] = {
            x: currentX,
            y: currentY
        };

        // Position visuell zuweisen
        draggable.style.left = `${currentX}px`;
        draggable.style.top = `${currentY}px`;
    }

    function event_pointerup(e: Event) {
        let draggable  = e.target as HTMLElement;
        let currentPos = rings_currentPos[draggable?.id];

        if(!draggable || !currentPos || locked)
            return;

        let currentX = currentPos.x;
        let currentY = currentPos.y;

        if (!isDragging)
            return;

        isDragging = false;
        draggable.releasePointerCapture(e.pointerId);

        let closestTarget = null;
        let minDistance = Infinity;

        // Berechne die Distanz zu jedem Ziel via Satz des Pythagoras
        targets.forEach(target => {
            const distance = Math.sqrt(
                Math.pow(currentX - target.x, 2) + Math.pow(currentY - target.y, 2)
            );
            
            if (distance < minDistance) {
                minDistance = distance;
                closestTarget = target;
            }
        });

        
        if(closestTarget && closestTarget.items.length && closestTarget.items[closestTarget.count - 1] < draggable.size)
            closestTarget = null;

        // Reset to start position
        if(closestTarget == null || minDistance > SNAP_THRESHOLD) {
            rings_currentPos[draggable?.id] = {
                x: (draggable.snappedTo ? draggable.snappedTo.x - 90 : 0),
                y: (draggable.snappedTo ? draggable.snappedTo.y : 0)
            };

            // Animation aktivieren und finale Position setzen
            draggable.style.left = resetX;
            draggable.style.top  = resetY;

            console.info('updated to', rings_currentPos[draggable?.id]);
        }

        // Wenn das nächste Ziel innerhalb der Snap-Reichweite liegt -> Einrasten!
        if (closestTarget && minDistance <= SNAP_THRESHOLD) {
            if(draggable.snappedTo) {
                draggable.snappedTo.count -= 1;
                draggable.snappedTo.items.pop();
            }

            let offset = closestTarget.count;
            currentX = closestTarget.x - 90;
            currentY = closestTarget.y - (offset * 42) + 250;
            
            rings_currentPos[draggable?.id] = {
                x: currentX,
                y: currentY
            };


            draggable.snappedTo = closestTarget;
            draggable.snappedIndex = offset;
            draggable.snappedTo.count += 1;
            draggable.snappedTo.items.push(draggable.size);

            // Animation aktivieren und finale Position setzen
            draggable.classList.add('snapping');
            draggable.style.left = `${currentX}px`;
            draggable.style.top  = `${currentY}px`;


            checkWinningCondition(draggable.snappedTo);
        }
    }
    

    function setRingToTarget(ring_id: string, target: any) {
        let draggable = document.getElementById(`${ring_id}`);

        if(!draggable)
            return;

        if(draggable.snappedTo) {
            draggable.snappedTo.count -= 1;
            draggable.snappedTo.items.pop();
        }

        let currentX  = target.x - 90;
        let currentY  = target.y - (target.count * 42) + 250;
        

        rings_currentPos[ring_id] = {
            x: currentX,
            y: currentY
        };

        draggable.snappedTo = target;
        draggable.snappedIndex = target.count;
        target.count += 1;
        target.items.push(draggable.size);

        // finale Position setzen
        draggable.style.left = `${currentX}px`;
        draggable.style.top  = `${currentY}px`;
    }



    function checkWinningCondition(target: any): boolean {
        if(!target || target.id != 'target_3')
            return false;

        if(target.items.length < RING_COUNT)
            return false;

        // optional check
        for(let i = 1; i < target.items.length; i++) {
            if(target.items[i] === undefined)
                return false;
            if(target.items[i - 1] === undefined)
                return false;

            if(target.items[i - 1] < target.items[i])
                return false;
        }

        // won
        alert('You won the Game 🥳');
        return true;
    }

    function searchPossiblePart(): number {
        let x = targets.map(t => t.items[t.count - 1]);
        x = x.sort().filter(x => x != undefined && x != 0);
        x = x[0];

        return (Number.isInteger(x) ? x : -1);
    }

    export async function solve() {
        if(locked || checkWinningCondition(targets[0]))
            return;

        locked = true;
        isDragging = false;

        let currentTarget;
        let currentRing;
        do {

            currentRing = rings[rings.length - 1];
            let i = targets.indexOf(currentRing.snappedTo) + 1;
            currentTarget = targets[(i >= targets.length ? 0: i)];

            setRingToTarget(currentRing.id, currentTarget);
            await delay(STEP_TIMEOUT);

            currentRing = rings[rings.length - 1 - searchPossiblePart()];
            if(currentRing) {
                i = targets.indexOf(currentRing.snappedTo) + 1;
                if(i >= targets.length)
                    i = 0;

                currentTarget = targets[i];
                if(currentTarget.items[currentTarget.items.length - 1] < currentRing.size) {
                    i++;
                    currentTarget = targets[(i >= targets.length ? 0: i)];
                }

                setRingToTarget(currentRing.id, currentTarget);
                await delay(STEP_TIMEOUT);
            }
        } while(!checkWinningCondition(currentTarget) && locked);

        locked = false;
    }

    export async function reset() {
        locked = false;

        // Alle Snap-Ziele aus dem DOM auslesen und ihre Koordinaten speichern
        targets = Array.from(document.querySelectorAll('.snap-target')).map((el, i) => {
            return {
                x: parseInt(el.style.left),
                y: parseInt(el.style.top),
                id: el.id,
                count: 0,
                items: [] as any[]
            };
        });

        // get ring elements
        rings = [];
        for(let i = 1; i <= RING_COUNT; i++) {
            let x = document.getElementById(`ring_${i}`);

            if(x) {
                x.snappedTo = null;
                rings.push(x);
                rings_currentPos[`ring_${i}`] = {
                    x: 0,
                    y: 0
                };
                x.addEventListener('pointerdown', event_pointerdown);
                x.addEventListener('pointermove', event_pointermove);
                x.addEventListener('pointerup', event_pointerup);


                x.size = (RING_COUNT - i);
                x.style.width = `${40 + ((RING_COUNT - i) * 30)}px`;
                x.style.marginLeft = `${(i * 15)}px`;

                setRingToTarget(`ring_${i}`, targets[0]);
            }
        }
    }
</script>


<style>
    /* Der Bereich, in dem sich alles abspielt */
    #container {
        position: relative;
        width: 100%;
        height: 880px;
        border: 2px dashed #ccc;
        background-color: #fff;
        border-radius: 8px;
        overflow: hidden;
    }

    /* Die vordefinierten "Snap-Zonen" (Dropzones) */
    .snap-target {
        position: absolute;
        margin-left: 30px;
        width: 40px;
        height: 280px;
        border: 2px dashed #aaa;
        background-color: rgba(0, 0, 0, 0.02);
        border-radius: 8px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        color: #888;
        pointer-events: none; /* Verhindert Interaktions-Konflikte beim Draggen */
    }

    /* Das ziehbare Element */
    .draggable {
        position: absolute;
        width: 160px;
        height: 40px;
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        border-radius: 8px;
        cursor: grab;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        touch-action: none; /* Wichtig für mobiles Draggen, verhindert Scrollen */
        user-select: none; /* Verhindert Textmarkierung beim Ziehen */
        transition: transform 0.1s ease;
    }

    .draggable:active {
        cursor: grabbing;
        transform: scale(1.05);
        box-shadow: 0 8px 15px rgba(0,0,0,0.15);
    }

    /* Sanfte Animation beim Einrasten */
    .snapping {
        transition: left 0.2s ease-out, top 0.2s ease-out !important;
    }
</style>


<div class="w-full h-full">

    <div id="container" bind:this={container} >
        <!-- Vordefinierte Snap-Positionen (über top/left gesteuert) -->
        <div id="target_1" class="snap-target" style="left:  90px; top: 90px;">Ziel 1</div>
        <div id="target_3" class="snap-target" style="left: 750px; top: 90px;">Ziel 3</div>
        <div id="target_2" class="snap-target" style="left: 430px; top: 570px;">Ziel 2</div>

        <!-- Das ziehbare Element (Standard-Startposition) -->
        {#each { length: RING_COUNT } as x, i}
            <div id={`ring_${i + 1}`} class="draggable bg-blue-500" style={`left: 20px; top: ${20 + (30 * i)}px;`}>{RING_COUNT - i}</div>        
        {/each}
    </div>

</div>